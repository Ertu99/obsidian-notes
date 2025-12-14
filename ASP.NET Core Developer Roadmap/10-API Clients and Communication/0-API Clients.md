  **API Clients and Communications**, mikroservis mimarisinin ve modern web uygulamalarının olmazsa olmazıdır. Çünkü günümüzde hiçbir uygulama bir ada değildir; ödeme için Stripe'a, harita için Google'a, hava durumu için Meteoroloji'ye bağlanmak zorundadır.

Attığın metin `HttpClient` sınıfını temel alarak özetlemiş. Ancak bir **Mimar** olarak bu konuyu; **Socket Exhaustion (Soket Tükenmesi)** problemi, **Resilience (Dayanıklılık)** desenleri ve **IHttpClientFactory** mekanizması üzerinden, yani "buzdağının görünmeyen kısmı" ile inceleyeceğiz.

---

### 1. Temel Araç: `HttpClient` ve Büyük Yalan

.NET dünyasında dışarıya HTTP isteği atmak için HttpClient sınıfı kullanılır.

Bu sınıf IDisposable arayüzünü uygular. C# derslerinde ne öğrendik? "IDisposable varsa using bloğu içine al."

**İşte burada büyük bir tuzak var.**

**Yanlış Kullanım (Junior Hatası):**

C#

```cs
using (var client = new HttpClient())
{
    var response = await client.GetAsync("https://api.google.com");
}
```

Mühendislik Analizi (Neden Yanlış?):

Sen using bloğundan çıktığında client nesnesi yok edilir (Dispose). ANCAK, arka planda açtığı TCP Bağlantısı (Socket) hemen kapanmaz. İşletim sistemi bu portu bir süre daha "beklemede" (TIME_WAIT state) tutar.

Eğer yüksek trafikli bir döngüde sürekli `new HttpClient()` yaparsan, sunucunun tüm portlarını tüketirsin (**Socket Exhaustion**). Uygulama "Bağlantı açamıyorum" diyerek çöker.

---

### 2. Çözüm: `IHttpClientFactory` (Fabrika Deseni)

Microsoft bu sorunu çözmek için .NET Core 2.1 ile **`IHttpClientFactory`** arayüzünü getirdi.

- **Mantık:** Sen fabrika'dan bir istemci istersin. Fabrika sana bir istemci verir ama arka plandaki TCP bağlantılarını (MessageHandler) bir **Havuzda (Pool)** yönetir.
    
- **Fayda:** Bağlantılar tekrar kullanılır (Re-use). Soket tükenmesi yaşanmaz. Ayrıca DNS değişiklikleri (örneğin Google'ın IP'si değişirse) otomatik algılanır (Singleton HttpClient'ın yapamadığı şey budur).
    

**Kullanım Türleri:**

1. **Basic Usage:** `_factory.CreateClient()` (Basit ama konfigürasyonsuz).
    
2. **Named Clients:** `builder.Services.AddHttpClient("Github", c => c.BaseAddress = ...)` (İsimle çağırırsın).
    
3. **Typed Clients (Best Practice):** Servis bazlı özelleştirme.
    

**Typed Client Örneği (Mühendislik Standartı):**

C#

```cs
// 1. Özel bir sınıf tanımla
public class GitHubService
{
    private readonly HttpClient _client;

    // Factory, HttpClient'ı konfigüre edip buraya enjekte eder (DI)
    public GitHubService(HttpClient client)
    {
        _client = client;
    }

    public async Task<string> GetUser(string username)
    {
        return await _client.GetStringAsync($"/users/{username}");
    }
}

// 2. Program.cs'te kaydet
builder.Services.AddHttpClient<GitHubService>(c => 
{
    c.BaseAddress = new Uri("https://api.github.com");
    c.DefaultRequestHeaders.Add("User-Agent", "MyApp");
});
```

---

### 3. Resilience (Dayanıklılık) ve Polly

Dış dünya güvenilmezdir. İnternet kesilir, karşı sunucu çöker, timeout olur.

Bir Mimar, "Hata olursa ne yapayım?" sorusunu kodlamalıdır.

.NET dünyasında bu işin standardı **Polly** kütüphanesidir. `IHttpClientFactory` ile entegre çalışır.

**Kritik Politikalar (Policies):**

1. **Retry (Yeniden Dene):** Hata alınca hemen pes etme, 3 kere daha dene.
    
    - _Mühendislik Notu:_ **Exponential Backoff** (Üstel Bekleme) kullan. İlk hatada 2sn, ikincide 4sn, üçüncüde 8sn bekle. (Karşı sunucuyu boğmamak için).
        
2. **Circuit Breaker (Devre Kesici):** Evdeki sigorta gibidir.
    
    - Eğer karşı taraf üst üste 50 kere hata verdiyse, 51. isteği gönderme! "Devreyi aç" ve doğrudan hata fırlat.
        
    - Belirli bir süre (örn: 1 dakika) bekle, sonra sistemi yavaşça tekrar dene (Half-Open).
        
    - _Amaç:_ Çökmüş sistemi istek yağmuruna tutup iyice öldürmemek (Fail Fast).
        

---

### 4. REST Client Kütüphaneleri (Refit)

Her API isteği için `_client.GetAsync`, `JsonSerializer.Deserialize`... yazmak bir süre sonra yorar (Boilerplate).

**Refit**, bu işi bir Interface tanımlayarak yapmanı sağlar. (Java dünyasındaki Retrofit gibi).

C#

```cs
// Sadece arayüz tanımlarsın
public interface IGitHubApi
{
    [Get("/users/{user}")]
    Task<User> GetUser(string user);
}

// Refit arka planda kodunu üretir. Sen sadece metot çağırırsın.
var user = await _gitHubApi.GetUser("octocat");
```

Kod okunabilirliği ve bakımı için muazzamdır.

---

### 5. Serileştirme (Serialization) Performansı

API'den gelen veri genelde **JSON** formatındadır. Bunu C# nesnesine çevirmek (Deserialization) CPU maliyetli bir iştir.

- **Newtonsoft.Json:** Eski kral. Çok yetenekli ama yavaştır ve çok bellek tüketir.
    
- **System.Text.Json:** Yeni kral. .NET Core ile geldi. Performans odaklıdır. `Span<T>` gibi düşük seviye bellek yapılarını kullanır.
    
- **Best Practice:** Mümkün olduğunca varsayılan `System.Text.Json` kullan.
    

---

