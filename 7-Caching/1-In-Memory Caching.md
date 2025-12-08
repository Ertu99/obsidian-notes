
Metinde de belirtildiği gibi, bu yöntem veriyi "bilgisayarın ana belleğinde (RAM)" saklar. Bu, erişim hızı açısından ulaşabileceğin **en yüksek performanstır** çünkü arada ne ağ trafiği (Redis gibi) ne de disk okuma (Veritabanı gibi) vardır.

Ancak, bir mühendis olarak bu gücü kullanırken dikkat etmen gereken **ölçeklenebilirlik (scalability)** ve **bellek yönetimi (memory pressure)** riskleri vardır.

---

### 1. Mimari: `IMemoryCache` Nedir?

ASP.NET Core'da In-Memory Cache, uygulamanın kendi işlemi (process) içinde yaşayan bir **Key-Value (Anahtar-Değer)** deposudur.

- **Nasıl Çalışır?** Arka planda aslında gelişmiş bir `ConcurrentDictionary` (Thread-Safe Sözlük) gibi çalışır.
    
- **Yaşam Döngüsü:** Uygulama (IIS veya Kestrel) kapanırsa veya yeniden başlarsa, **RAM'deki tüm veri silinir.** (Volatile).
    
- **Kullanım:** Dependency Injection ile `IMemoryCache` arayüzünü enjekte ederek kullanırız.
    

---

### 2. Mühendislik Riski 1: Sticky Sessions (Yapışkan Oturumlar)

Eğer uygulamanı tek bir sunucuda değil de, yük dengeleyici (Load Balancer) arkasında **birden fazla sunucuda (Web Farm)** çalıştırıyorsan, In-Memory Cache büyük bir tuzağa dönüşür.

- **Senaryo:**
    
    1. Kullanıcı A, **Sunucu-1**'e düştü. Haberleri çekti ve Sunucu-1'in RAM'ine yazıldı.
        
    2. Kullanıcı A F5'e bastı, Load Balancer onu **Sunucu-2**'ye gönderdi.
        
    3. **Sorun:** Sunucu-2'nin RAM'i boş! Veri yok. Kullanıcı tekrar veritabanına gitmek zorunda kalır.
        

Daha kötüsü veri tutarlılığıdır: Sunucu-1 veriyi günceller, Sunucu-2 eski veriyi göstermeye devam eder.

**Çözüm:** Çoklu sunucu mimarisinde In-Memory Cache kullanılmaz (veya çok dikkatli kullanılır). Bunun yerine **Distributed Cache (Redis)** kullanılır. Eğer illaki In-Memory kullanacaksan, Load Balancer ayarlarından **Sticky Sessions** (Kullanıcıyı hep aynı sunucuya yapıştır) özelliğini açmalısın.

---

### 3. Kod Pratiği: Cache-Aside Pattern Uygulaması

Modern .NET Core'da veriyi güvenli bir şekilde çekmek için `TryGetValue` veya `GetOrCreate` kullanılır.

C#

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;

    public ProductService(IMemoryCache cache)
    {
        _cache = cache;
    }

    public List<Product> GetProducts()
    {
        // 1. Önce Cache'e bak
        if (!_cache.TryGetValue("tum_urunler", out List<Product> products))
        {
            // 2. Cache Miss (Yoksa): Veritabanından çek
            products = _db.Products.ToList();

            // 3. Cache Ayarlarını Yap (Expiration)
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetAbsoluteExpiration(TimeSpan.FromMinutes(10)) // 10 dk ömür
                .SetSlidingExpiration(TimeSpan.FromMinutes(2))   // 2 dk kullanılmazsa sil
                .SetPriority(CacheItemPriority.Normal);          // RAM dolarsa öncelik

            // 4. Cache'e Yaz (Set)
            _cache.Set("tum_urunler", products, cacheOptions);
        }

        return products;
    }
}
```

---

### 4. Mühendislik Riski 2: Memory Leak (Bellek Sızıntısı)

Metinlerde "geçici depolama" dense de, eğer dikkat etmezsen sunucuyu çökertirsin.

**Sorun:** RAM sınırsız değildir. Eğer Cache'e attığın verileri (özellikle Key'leri) kontrolsüzce dinamik yaparsan (örn: `_cache.Set($"User_{Guid.NewGuid()}", data)`) ve bunlara süre (Expiration) koymazsan, RAM dolar ve uygulama **OutOfMemoryException** vererek kapanır.

**Çözüm:**

1. **Expiration:** Her cache kaydına mutlaka bir ömür biç.
    
2. **Size Limit:** Cache servisini eklerken limit koyabilirsin.
    
    C#
    
    ```csharp
    builder.Services.AddMemoryCache(options =>
    {
        options.SizeLimit = 1024; // Maksimum 1024 birim sakla
    });
    ```
    
    _Not: SizeLimit kullanıyorsan, her `Set` işleminde o verinin boyutunu (`.SetSize(1)`) belirtmek zorundasın._
    

---

### 5. Kritik Konu: Expiration Stratejileri

Bir önceki derste bahsetmiştik ama kod tarafında nasıl yönetildiğine bakalım. İki tür süre vardır ve bunları **birlikte** kullanmak en iyi pratiktir.

1. **Absolute Expiration (Mutlak):** "Ne olursa olsun 1 saat sonra sil." (Bayat veriyi önler).
    
2. **Sliding Expiration (Kayar):** "Erişildiği sürece ömrünü 10 dakika uzat."
    

Tehlike: Sadece Sliding kullanırsan ve veri sürekli okunursa, veri sonsuza kadar RAM'de kalır ve bayatlar (Immortal Data).

Best Practice: SetSlidingExpiration(TimeSpan.FromMinutes(10)).SetAbsoluteExpiration(TimeSpan.FromHour(1)) -> Sürekli erişilse bile en fazla 1 saat yaşasın.

---

### 6. Thread Safety ve Race Condition

`IMemoryCache` kendi içinde **Thread-Safe**'tir. Yani aynı anda iki farklı thread `Set` veya `Get` yapabilir, cache bozulmaz.

Ancak (Büyük Tuzak):

Cache'in içinde sakladığın nesne (List<Product>) Thread-Safe olmayabilir!

- Thread A: Cache'ten listeyi aldı, içine ürün ekliyor.
    
- Thread B: Cache'ten aynı listeyi (referansı) aldı, içinden ürün siliyor.
    
- **Sonuç:** Uygulama hatası (Collection modified).
    

**Çözüm:** Cache'ten veriyi aldıktan sonra üzerinde değişiklik yapacaksan, o nesneyi değiştirmemeli (Immutable) veya kopyasını almalısın.

---

