> N-Tier (katmanlı mimari), bir uygulamayı görevlerine göre farklı katmanlara (layer’lara) ayırma yaklaşımıdır.

Amaç:

✅ Kodun **düzenli**, **yeniden kullanılabilir** ve **bakımı kolay** olmasıdır.

✅ Her katman kendi sorumluluğunu taşır.

---

## 🧱 2️⃣ “N” ne demek?

- Genelde 3 veya 4 katmanlı yapı kullanılır:
    
    - “N” → **Number of tiers (katman sayısı)** anlamına gelir.
    
    👉 **3-Tier Architecture** en yaygın olanıdır.
    

---

## 🧩 3️⃣ 3-Tier (3 Katmanlı) Mimari

|Katman|Görevi|Örnek|
|---|---|---|
|**1. Presentation Layer (Sunum Katmanı)**|Kullanıcı ile etkileşir. API veya UI katmanıdır.|[ASP.NET](http://ASP.NET) MVC Controller, React, Angular, Blazor|
|**2. Business Logic Layer (İş Katmanı)**|İş kuralları ve işlemler burada yapılır.|“HotelService” gibi sınıflar|
|**3. Data Access Layer (Veri Erişim Katmanı)**|Veritabanı ile iletişimi sağlar.|Repository, EF Core Context|

---

### 📘 Katmanlar arası akış (örnek .NET senaryosu):

```
[Frontend / API] → [Business Layer] → [Data Access Layer] → [Database]

```

---

## 🧠 4️⃣ Gerçek Örnek (HotelBooking API)

### 🔹 1. Presentation (WebAPI Katmanı)

Kullanıcı isteğini alır ve Servis’e iletir.

```csharp
[ApiController]
[Route("api/[controller]")]
public class HotelController : ControllerBase
{
    private readonly IHotelService _hotelService;

    public HotelController(IHotelService hotelService)
    {
        _hotelService = hotelService;
    }

    [HttpGet("list")]
    public async Task<IActionResult> GetAll()
    {
        var hotels = await _hotelService.GetAllAsync();
        return Ok(hotels);
    }
}

```

---

### 🔹 2. Business Layer (Application Katmanı)

İş kurallarını içerir.

```csharp
public class HotelService : IHotelService
{
    private readonly IHotelRepository _repository;

    public HotelService(IHotelRepository repository)
    {
        _repository = repository;
    }

    public async Task<List<Hotel>> GetAllAsync()
    {
        // Burada örneğin bir filtreleme kuralı eklenebilir
        return await _repository.GetAllAsync();
    }
}

```

---

### 🔹 3. Data Access Layer (Infrastructure Katmanı)

Veritabanına erişim sağlar.

```csharp
public class HotelRepository : IHotelRepository
{
    private readonly AppDbContext _context;

    public HotelRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<List<Hotel>> GetAllAsync()
    {
        return await _context.Hotels.AsNoTracking().ToListAsync();
    }
}

```

---

## ⚙️ 5️⃣ Katmanların Görev Ayrımı

|Katman|Açıklama|Sorumluluk|
|---|---|---|
|**Presentation (UI / API)**|Kullanıcı veya istemciyle iletişim kurar.|Request alır, Response döner.|
|**Business (Service)**|İş kurallarını uygular.|Validasyon, hesaplama, koşul kontrolü.|
|**Data Access (Repository / DAL)**|Veritabanıyla konuşur.|CRUD işlemleri yapar.|

---

## 🧩 6️⃣ 4-Tier veya daha fazlası?

Bazen 4 veya 5 katmanlı yapı da kurulur:

- **4-Tier:** + `Core` (Entities, Interfaces, DTO’lar)
- **5-Tier:** + `Infrastructure` (EmailService, Logging, Caching vs.)

Yani:

```
API → Application → Domain/Core → Infrastructure → Database

```

---

## 🎯 7️⃣ Mülakat cevabı (kısa ve net)

> “N-Tier mimari, uygulamayı görevlerine göre katmanlara ayıran yapıdır.
> 
> Genellikle Presentation, Business ve Data Access katmanlarından oluşur.
> 
> Böylece kod daha düzenli, test edilebilir ve bakımı kolay hale gelir.”