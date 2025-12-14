## 🎯 Tanım:

**DTO (Data Transfer Object)**,

farklı katmanlar (örneğin API ↔ Service ↔ Database) arasında veri taşımak için kullanılan **taşıyıcı model**dir.

Yani:

> DTO, sadece veri taşır,
> 
> **iş mantığı veya veritabanı bağlantısı** içermez.

---

## 🔹 Amaç:

Bir entity’yi (örneğin `Hotel`) doğrudan API’de kullanmak istemeyiz çünkü:

- Tüm alanları dışarı açmak **güvenlik riski** oluşturur,
- Veritabanı modelini doğrudan dış dünyaya açmak **bağımlılık** yaratır,
- API’ye gelen verilerle **doğrulama** yapmak zorlaşır.

Bu yüzden **entity’nin sade, güvenli bir kopyasını** (DTO’sunu) kullanırız.

---

## 🧱 Örnek:

### Entity (veritabanı modeli)

```csharp
public class Hotel
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public string City { get; set; } = null!;
    public int Star { get; set; }
    public ICollection<Room> Rooms { get; set; } = new List<Room>();
}

```

### CreateHotelDto (API’den gelen veri)

```csharp
public class CreateHotelDto
{
    public string Name { get; set; } = null!;
    public string City { get; set; } = null!;
    public int Star { get; set; }
}

```

### HotelDto (API’den dönen veri)

```csharp
public class HotelDto
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public string City { get; set; } = null!;
    public int Star { get; set; }
}

```

---

## 🧭 Neden Kullanırız?

|Neden|Açıklama|
|---|---|
|🛡️ **Güvenlik**|Veritabanı alanlarını doğrudan dışarı açmazsın.|
|⚙️ **Ayrık katmanlar**|Entity değişse bile API kırılmaz.|
|🧰 **Validasyon kolaylığı**|`[Required]`, `[StringLength]` gibi kurallar DTO üzerinde yapılır.|
|🔄 **Veri kontrolü**|Hangi alanların kullanıcıdan alınacağını netleştirir.|
|📦 **Daha temiz API**|Swagger ve dokümantasyonda net veri yapısı görünür.|

---

## 📊 DTO Türleri

|DTO Tipi|Kullanım|Örnek Endpoint|
|---|---|---|
|**Create DTO**|Yeni kayıt eklemek için|`POST /api/hotels`|
|**Update DTO**|Var olan kaydı güncellemek için|`PUT /api/hotels/{id}`|
|**Read DTO (Response DTO)**|Dış dünyaya veri döndürmek için|`GET /api/hotels`|

---