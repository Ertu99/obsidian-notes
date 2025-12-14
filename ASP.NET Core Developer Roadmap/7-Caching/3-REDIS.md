
Redis (Remote Dictionary Server), sadece bir "Cache" değildir. O, **In-Memory Data Structure Store** (Bellek İçi Veri Yapıları Deposu) olarak tanımlanır.

Bir Backend Mühendisi olarak Redis'i anlamak için onu Memcached veya diğer basit Key-Value depolarından ayıran şu özelliğini bilmelisin: Redis, verinin "tipini" bilir.

Sadece string (metin) saklamaz; Listeler, Kümeler, Hash'ler saklar ve bunlar üzerinde atomik işlemler yapabilir.

Bu konuyu, .NET dünyasında nasıl kullanıldığını ve **Single-Threaded** (Tek İş Parçacıklı) mimarisinin getirdiği kritik riskleri inceleyerek ele alacağız.

---

### 1. Redis'in Süper Güçleri: Veri Yapıları

Bir C# geliştiricisi olarak `List<T>`, `Dictionary<K,V>`, `HashSet<T>` gibi yapıları bilirsin. Redis, bu yapıların aynısını RAM üzerinde tutar ve tüm sunucularının kullanımına açar.

#### A. Strings (Metinler) - Klasik Cache

En temel yapıdır.

- **Kullanım:** JSON serileştirilmiş nesneler (`User_1: "{...}"`), HTML parçaları veya Sayaçlar.
    
- **Özel Yetenek:** `INCR` komutu. C# tarafında `lock` kullanmadan güvenli bir şekilde sayaç artırabilirsin (Atomic Increment).
    
    - _Örnek:_ Site görüntülenme sayısı.
        

#### B. Lists (Listeler) - Kuyruk Sistemi

Bağlı liste (Linked List) mantığıyla çalışır.

- **Kullanım:** Basit mesaj kuyrukları (Queue).
    
- **Komutlar:** `LPUSH` (Sola ekle), `RPOP` (Sağdan al). RabbitMQ kurmak istemediğin küçük işlerde harika bir kuyruk sistemidir.
    

#### C. Sets (Kümeler) - Benzersizlik

İçine ne atarsan at, tekrar edenleri siler (Unique).

- **Kullanım:** "Şu an sitede aktif olan IP adresleri". Aynı IP 100 kere gelse de Set içinde 1 kere durur. Ayrıca Kesişim/Birleşim (`INTERSECT`, `UNION`) işlemlerini sunucu tarafında çok hızlı yapar.
    

#### D. Sorted Sets (Sıralı Kümeler) - Liderlik Tablosu

Her elemanın bir "Skoru" vardır. Redis bunları otomatik sıralar.

- **Kullanım:** Oyunlardaki "Leaderboard". "En çok puan alan ilk 10 kişiyi getir" (`ZRANGE 0 9`) komutu mikrosaniyeler sürer. Bunu SQL ile yapmaya çalışırsan (`ORDER BY Score DESC LIMIT 10`) milyonlarca kayıtta sunucu ağlar. 


#### E. Hashes (Sözlükler)

Bir anahtar altında birden fazla alan (Field-Value) tutar.

- **Kullanım:** Kullanıcı profili. `User:100` anahtarının altında `Name: Ahmet`, `Age: 26` alanlarını tutabilirsin.
    
- **Avantajı:** Sadece yaşı güncellemek için tüm nesneyi çekip-serileştirip-tekrar yazmana gerek yoktur. Sadece `HSET User:100 Age 27` dersin.
    

---

### 2. Kalıcılık (Persistence): RAM Uçarsa Ne Olur?

Redis bellek tabanlıdır, yani sunucunun fişi çekilirse veri gider. AMA Redis'i bir veritabanı gibi kullanmak istersen iki seçeneğin vardır:

1. **RDB (Snapshot):** Belirli aralıklarla (örn: her 5 dakikada bir) RAM'in fotoğrafını çeker ve diske kaydeder.
    
    - _Risk:_ Elektrik kesilirse son 5 dakikalık veriyi kaybedersin.
        
2. **AOF (Append Only File):** Gelen her yazma komutunu (`SET`, `INCR`) bir log dosyasına yazar.
    
    - _Güvenlik:_ Veri kaybı neredeyse sıfırdır ama disk kullanımı artar ve biraz daha yavaştır.
        

**Mühendislik Kararı:** Eğer Redis'i sadece **Cache** olarak kullanıyorsan (veri zaten veritabanında varsa), persistence özelliklerini kapatıp performansı artırabilirsin.

---

### 3. Kritik Mimari Bilgi: Single-Threaded Yapı

Redis, (bazı yan işlemler hariç) komutları işlemek için Tek Bir CPU Çekirdeği kullanır.

Bu ne demek?

- Redis saniyede 100.000 işlemi sıraya dizer ve teker teker yapar.
    
- Eğer sen (veya yazdığın kod) Redis'e **çok uzun süren bir komut** gönderirse, o komut bitene kadar **arkadaki 10.000 istek beklemek zorundadır.**
    

Ölümcül Komut: KEYS *

Bu komut "Bana veritabanındaki tüm anahtarları listele" der. Eğer içeride 10 milyon anahtar varsa, Redis bunu tararken 1-2 saniye donar. O 1-2 saniye boyunca siten kilitlenir (Timeouts). Bu yüzden Production ortamında KEYS komutu yasaklanır, yerine SCAN kullanılır.

---

### 4. .NET Entegrasyonu: `StackExchange.Redis`

.NET dünyasında Redis ile konuşmak için standart kütüphane **StackExchange.Redis**'tir (Stack Overflow ekibi tarafından yazılmıştır).

Bu kütüphane **Multiplexer** desenini kullanır. Yani Redis ile tek bir TCP bağlantısı açar ve tüm thread'ler bu bağlantıyı paylaşır.

**Kod Örneği (Connection):**

C#

```csharp
// Singleton olarak kaydedilmeli!
var multiplexer = ConnectionMultiplexer.Connect("localhost");
IDatabase db = multiplexer.GetDatabase();

// Yazma
db.StringSet("anahtar", "değer");

// Okuma
string deger = db.StringGet("anahtar");
```

---

### 5. Pub/Sub (Yayınla/Abone Ol)

Metinde "message broker" olarak da geçtiği gibi, Redis basit bir haberleşme sistemi olarak da kullanılır.

- **Publisher:** "Haberler" kanalına bir mesaj atar.
    
- **Subscriber:** "Haberler" kanalını dinleyen 50 farklı servis varsa, hepsi bu mesajı anında alır.
    
- _Kullanım:_ Chat uygulamaları veya "Ayarlar değişti, cache'leri temizleyin" sinyali göndermek için.
    

---
### 1. Distributed Caching (Redis)

**🧒 6 Yaşındaki Çocuğa (Sınıf Tahtası Analojisi):** "Hatırlıyor musun, az önce boya kalemlerin sıranın üzerindeydi (In-Memory). Ama şimdi sınıf çok kalabalıklaştı ve öğretmen seni sürekli başka sıralara oturtuyor (Load Balancer). Kalemlerini sürekli yanında taşıyamazsın. Bunun yerine sınıfın ortasına kocaman bir **Ortak Dolap (Redis)** koyuyoruz. Herkes kalemini o dolaba koyuyor. Hangi sırada oturursan otur, kalemin lazım olduğunda koşup dolaptan alıyorsun. Ama küçük bir sorun var: Dolaba kalemi öylece fırlatamazsın. Onu güzelce kutusuna koyup, kilitleyip öyle yerleştirmelisin (**Serialization**). Kullanacağın zaman da kutuyu açmalısın (**Deserialization**). Bu kutulama işi biraz vaktini alır ama en azından kalemlerin her zaman güvendedir ve tüm arkadaşların aynı dolabı kullanabilir."

**👨‍💼 Mülakatta Yöneticiye (Abstraction):** "Distributed Cache, özellikle **Mikroservis** ve **Container (Docker/Kubernetes)** mimarilerinde veri tutarlılığını (Data Consistency) sağlamak için kullandığımız, uygulamadan bağımsız çalışan merkezi bir önbellek yapısıdır. Uygulamamız yatayda ölçeklendiğinde (Horizontal Scaling), sunucuların birbiriyle senkronize olması için In-Memory yerine **Redis** gibi harici bir çözüm zorunludur. Ancak bir Mimar olarak burada şu maliyetin farkındayım:

1. **Network Latency:** Veriye erişim, In-Memory gibi nanosaniyelerle değil, ağ üzerinden olduğu için milisaniyelerle ölçülür.
    
2. **Serialization Overhead:** Veriyi Redis'e yazarken Binary/JSON formatına çevirmek (Serialize) ve okurken geri çevirmek (Deserialize) ciddi bir CPU maliyeti yaratır. Bu yüzden Distributed Cache kullanırken nesne boyutlarını optimize etmeye ve gereksiz trafikten kaçınmaya özen gösteririm."