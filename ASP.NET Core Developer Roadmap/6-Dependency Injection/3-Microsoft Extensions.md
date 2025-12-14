 
Bu kütüphane, sadece ASP.NET Core'un bir parçası değildir; Konsol uygulamalarından Azure Functions'a kadar her yerde çalışan, bağımsız ve hafif (lightweight) bir Container'dır.

Çoğu geliştirici sadece `builder.Services.AddScoped(...)` yazar geçer. Ama bir **Backend Mühendisi**, bu kütüphanenin yeteneklerini (Open Generics, Factory Pattern) ve sınırlarını (neden bazen Autofac'e ihtiyaç duyduğumuzu) bilmek zorundadır.

Gel bu motoru parçalarına ayıralım.

---

### 1. Mimari: İki Dev Sütun

Bu kütüphane iki temel arayüz üzerine kuruludur. Bunları birbirine karıştırmamak hayati önem taşır.

#### A. `IServiceCollection` (Mimar / Planlama Masası)

Uygulama henüz başlamadan önce (`Program.cs`), kimin kim olduğunu tanımladığımız listedir.

- **Görevi:** Kayıt (Registration).
    
- **Yapısı:** Aslında basit bir `List<ServiceDescriptor>` koleksiyonudur. Her `AddScoped` dediğinde bu listeye bir satır eklersin.
    
- **Analoji:** Restoran menüsü. Hangi yemeğin içinde ne var, nasıl yapılır burada yazar.
    

#### B. `IServiceProvider` (Fabrika / Üretim Hattı)

Uygulama çalışmaya başladığında (`app.Run()`), planları alıp gerçeğe dönüştüren yapıdır.

- **Görevi:** Çözümleme (Resolution).
    
- **Yapısı:** İmmutable (Değiştirilemez). Uygulama çalışırken yeni servis ekleyemezsin.
    
- **Analoji:** Mutfak şefi. Sipariş gelince menüye bakar ve yemeği yapar.
    

---

### 2. Kayıt Yöntemleri (Registration Patterns)

Sadece `AddScoped<IA, A>()` bilmek yetmez. Mühendislik gerektiren durumlarda şu yöntemleri kullanırsın:

#### A. Factory Pattern (Fabrika Yöntemi)

Bazen bir nesneyi oluşturmak için `new` demek yetmez, özel bir ayar yapman gerekir.

C#

```csharp
// Container'a diyoruz ki: "Biri senden IDatabase isterse, şu fonksiyonu çalıştır"
services.AddScoped<IDatabase>(provider => 
{
    var config = provider.GetRequiredService<IConfiguration>();
    var connectionString = config["DbConnection"];
    
    // Nesneyi manuel konfigüre edip dönüyoruz
    return new SqlDatabase(connectionString, timeout: 30);
});
```

#### B. Open Generics (Açık Generik Tipler)

Her entity için tek tek `UserRepository`, `ProductRepository` kaydetmek ameleliktir.

C#

```csharp
// "IRepository<T>" isteyen herkese "Repository<T>" ver.
// T ne olursa olsun (User, Product) otomatik eşleşir.
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

#### C. `TryAdd` (Kütüphane Yazarları İçin)

Eğer bir kütüphane yazıyorsan ve kullanıcının senin servisini ezmesine izin vermek istiyorsan:

C#

```csharp
// "Eğer IEmailService daha önce kaydedilmemişse, benimkini kaydet.
// Ama kullanıcı kendi servisini kaydettiyse, ona dokunma."
services.TryAddScoped<IEmailService, DefaultEmailService>();
```

---

### 3. Çözümleme: `GetService` vs `GetRequiredService`

Bu ikisi arasındaki fark, hata yönetimi stratejini belirler.

1. **`GetService<T>()`:**
    
    - Servis bulunamazsa **`null`** döner.
        
    - **Risk:** Eğer null kontrolü yapmazsan, kodun ilerleyen satırlarında `NullReferenceException` alırsın ve hatanın kaynağını (servisin eksik olduğunu) anlamak zorlaşır.
        
2. **`GetRequiredService<T>()` (Önerilen):**
    
    - Servis bulunamazsa anında **`InvalidOperationException`** fırlatır: _"No service for type 'T' has been registered."_
        
    - **Mühendislik Notu:** Hata Fail-Fast (Hızlı Patlama) prensibine uyar. Sorunu kaynağında yakalarsın. Her zaman bunu kullanmaya çalış.
        

---

### 4. Çoklu İmplementasyon (Strategy Pattern)

Mülakatların favori sorusudur: _"Aynı Interface'i miras alan 3 farklı sınıfı Container'a eklersen ne olur?"_

C#

```csharp
services.AddTransient<INotification, EmailNotification>();
services.AddTransient<INotification, SmsNotification>();
services.AddTransient<INotification, PushNotification>();
```

Bu kod hata vermez. Hepsi listeye eklenir.

- **Nasıl Çağırırım?**
    
    - Eğer constructor'da `INotification` istersen, **en son eklenen** (PushNotification) gelir.
        
    - Eğer hepsini birden istiyorsan, constructor'da **`IEnumerable<INotification>`** istemelisin. Container sana 3 servisi de içeren bir liste verir.
        

Bu özellik, **Strategy Pattern** uygulamak için muazzamdır. Bir döngüyle hepsini çalıştırabilirsin (Örn: Hem mail, hem SMS, hem Push at).

---

### 5. `IDisposable` Yönetimi (Çöp Yönetimi)

Mühendislik kuralı: **"Yaratan, yok eder."**

- Container ile oluşturduğun (`AddScoped`, `AddTransient`) ve `IDisposable` arayüzünü uygulayan (yani bellek temizliği gereken) nesneler, Container'ın sorumluluğundadır.
    
- İstek bittiğinde (Scope kapandığında), Container otomatik olarak o scope içinde yarattığı tüm disposable nesnelerin `.Dispose()` metodunu çağırır.
    
- **Risk:** Eğer `AddSingleton` olarak kaydettiğin bir servisin içine `IDisposable` bir nesne verirsen, o nesne uygulama kapanana kadar dispose edilmez (Memory Leak riski).
    

---

### 6. Scope Validation (Görünmez Kalkan)

Önceki derslerde "Singleton içine Scoped enjekte edersen patlar (Captive Dependency)" demiştik. Peki bunu nasıl fark ederiz?

ASP.NET Core, Development modunda çalışırken (ASPNETCORE_ENVIRONMENT=Development), Container arka planda ekstra bir kontrol yapar: Scope Validation.

Eğer yanlışlıkla Singleton içine Scoped bir servis verirsen, uygulama ayağa kalkarken sana bağırır ve hatayı gösterir.

Production modunda ise performans artsın diye bu kontrol kapanır. Bu yüzden hataları Development ortamında yakalamak çok önemlidir.

---

