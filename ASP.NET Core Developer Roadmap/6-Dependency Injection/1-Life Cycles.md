

Önceki derste DI'ın "neden" var olduğunu konuşmuştuk. Şimdi ise DI Container'a (Konteyner) verdiğimiz talimatların **bellek yönetimi (Memory Management)** ve **eşzamanlılık (Concurrency)** üzerindeki etkilerini mühendislik derinliğinde inceleyeceğiz. Bu konu, uygulamanın ne kadar RAM tüketeceğini ve çoklu kullanıcı altında hata verip vermeyeceğini belirler.

Attığın metinleri baz alarak 3 temel döngüyü ve özel durumları analiz edelim.

---

### 1. Transient (Her Seferinde Yeni)

Metinde dendiği gibi: _"A new instance of the object is created every time it is requested."_.

- **Çalışma Mantığı:** Sen veya uygulama içindeki başka bir servis, bu nesneyi her istediğinde, DI Container `new Class()` diyerek sıfır kilometre bir nesne oluşturur.
    
- **Kullanım Alanı:** Metin harika bir ipucu vermiş: _"Services that are stateless and do not need to maintain any data."_. Yani durum tutmayan, sadece hesap yapıp sonucu dönen servisler.
    
    - _Örnek:_ Bir vergi hesaplama servisi (`CalculatorService`). Girdi bellidir, çıktı bellidir. Hafızada bir şey saklamasına gerek yoktur.
        
- **Mühendislik Analizi (Memory Churn):** Transient nesneler çok sık oluşturulur ve işi bitince hemen Garbage Collector (Çöpçü) tarafından temizlenir. Bu durum bellekte sürekli bir "yap-boz" trafiği yaratır. Çok ağır (bellekte çok yer kaplayan) nesneleri Transient yapmak, sunucunun işlemcisini yorabilir.
    

### 2. Scoped (İstek Bazlı / Kapsamlı)

Web geliştiricilerinin en çok kullandığı türdür. Metinde şöyle tanımlanmış: _"Creates a new instance of an object for each unique request, but reuses the same instance for the same request."_.

- **Çalışma Mantığı:** Bir HTTP isteği (Request) sunucuya ulaştığında sanal bir "Kapsam" (Scope) oluşur. Bu istek bitene kadar, DI Container kime verirse versin, o nesnenin **aynı kopyasını** verir. İstek bittiği an, nesne imha edilir (Dispose).
    
- **Kullanım Alanı:** Metin nokta atışı yapmış: _"Request-scoped database context."_. Yani `DbContext`.
    
    - _Neden?_ Bir istek boyunca yapılan tüm veritabanı işlemleri (Sipariş ekle, Stoğu düş, Log at) aynı Transaction (İşlem) içinde olmalıdır. Bunu sağlamanın yolu, hepsinin aynı `DbContext` nesnesini kullanmasıdır.
        
- **Mühendislik Analizi (Isolation):** Scoped, verilerin birbirine karışmasını engeller. Ahmet'in isteği için oluşturulan nesneye Mehmet erişemez. Metinde buna _"Prevent cross-request contamination"_ denmiş.
    

### 3. Singleton (Tekil / Global)

En riskli ama en güçlü olandır. Metinde: _"Creates a single instance of an object and reuses it throughout the lifetime of the application."_.

- **Çalışma Mantığı:** Uygulama ilk ayağa kalktığında (veya ilk çağrıldığında) bir kere oluşturulur. Sunucu kapanana kadar RAM'de yaşar. 1 milyon kullanıcı da gelse hepsi **aynı bellek adresindeki** nesneyi kullanır.
    
- **Kullanım Alanı:** _"Services that need to maintain state or shared data across requests, such as a service that caches data."_.
    
    - _Örnek:_ `MemoryCache` servisi. Herkes aynı önbelleğe erişmeli ki veri paylaşılabilsin.
        
- **Mühendislik Riski (Thread Safety):** Singleton nesneler **Thread-Safe (İş Parçacığı Güvenli)** olmak zorundadır. Eğer Singleton bir servisin içinde `List<string>` varsa ve iki kullanıcı aynı anda bu listeye ekleme yaparsa uygulama patlar. Bu yüzden Singleton servislerde `lock` mekanizmaları veya `ConcurrentDictionary` gibi özel yapılar kullanılır.
    

### 4. Custom Lifecycle (Özel Döngü)

Metinde çok kısa değinilmiş ama ileri seviye bir konudur: _"You can also create a custom lifecycle by implementing the IServiceScopeFactory interface."_.

- **Ne Zaman Gerekir?** Örneğin, uygulamanın içinde HTTP isteğinden bağımsız, arka planda her 15 dakikada bir çalışan bir "Raporlama Botu" (Background Service) yazdın. Bu botun kendi "Scope"una ihtiyacı vardır. HTTP isteği olmadığı için standart Scoped çalışmaz. Burada manuel olarak `scopeFactory.CreateScope()` diyerek kendi yaşam alanını yaratırsın.
    


### Mühendislik Tuzağı: Captive Dependency (Esir Bağımlılık)

Bu konu Senior seviyesidir. Yaşam döngülerini birbirine karıştırırsan ne olur?

**Kural:** Uzun ömürlü bir servis, kısa ömürlü bir servisi tutamaz.

- **Singleton**, **Scoped** bir servise bağımlı olamaz (Constructor'ında isteyemez).
    

Neden?

Singleton bir kere üretilir ve sonsuza kadar yaşar. Eğer içine Scoped bir servis (örn: DbContext) alırsan, o DbContext'i de kendisiyle birlikte sonsuza kadar yaşatır (Esir alır).

Halbuki DbContext her istekte yenilenmeliydi. Sonuç: Veritabanı bağlantısı şişer, hafıza dolar ve uygulama çöker.

**Doğru Hiyerarşi:**

- Transient -> Her şeyi kullanabilir.
    
- Scoped -> Scoped ve Singleton kullanabilir.
    
- Singleton -> Sadece Singleton kullanabilir.
    

---

### Özet Tablo

| **Yaşam Döngüsü** | **Oluşturulma Sıklığı** | **Paylaşım Durumu**     | **Bellek Tüketimi** | **Örnek Kullanım**       |
| ----------------- | ----------------------- | ----------------------- | ------------------- | ------------------------ |
| **Transient**     | Her çağrıda (`new`)     | Asla paylaşılmaz        | Düşük (Kısa ömürlü) | Hesaplama araçları       |
| **Scoped**        | Her HTTP isteğinde      | İstek içinde paylaşılır | Orta                | Veritabanı (`DbContext`) |
| **Singleton**     | Uygulama ömründe 1 kez  | Herkesle paylaşılır     | Yüksek (Kalıcı)     | Cache, Config            |
