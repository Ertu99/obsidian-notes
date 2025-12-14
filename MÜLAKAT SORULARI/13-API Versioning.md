## 🎯 Neden versiyonlama?

Gerçek dünyada API’ler sürekli gelişir:

Yeni özellikler eklenir, bazı endpointler değişir veya kaldırılır.

Ama eski mobil/ web uygulamalar **hala eski endpointleri** kullanıyor olabilir.

Bu durumda eski client’ları kırmamak için API’ye versiyon eklenir:

```
/api/v1/hotels
/api/v2/hotels

```

---

## 🧠 Temel Mantık

Her versiyon kendi controller’ına veya action’ına yönlendirilir.

.NET Core’ta bu işi kolayca yapmak için

📦 `Microsoft.AspNetCore.Mvc.Versioning` paketini kullanırız.

---

## ⚙️ Kurulum

```
dotnet add package Microsoft.AspNetCore.Mvc.Versioning

```

---

## 🔧 Program.cs ayarları

```csharp
builder.Services.AddApiVersioning(options =>
{
    // Default versiyon
    options.DefaultApiVersion = new ApiVersion(1, 0);

    // Versiyon belirtilmezse defaultu kullan
    options.AssumeDefaultVersionWhenUnspecified = true;

    // Versiyon bilgisi response header'ına eklensin
    options.ReportApiVersions = true;

    // Versiyon belirleme yöntemleri (query / header / route)
    options.ApiVersionReader = new Microsoft.AspNetCore.Mvc.Versioning.UrlSegmentApiVersionReader();
});

```

---

## 🧱 Controller versiyonlama

```csharp
using Microsoft.AspNetCore.Mvc;

namespace HotelBooking.Api.Controllers.v1
{
    [ApiController]
    [ApiVersion("1.0")]
    [Route("api/v{version:apiVersion}/[controller]")]
    public class HotelsController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get() => Ok("V1 - Hotels list");
    }
}

namespace HotelBooking.Api.Controllers.v2
{
    [ApiController]
    [ApiVersion("2.0")]
    [Route("api/v{version:apiVersion}/[controller]")]
    public class HotelsController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get() => Ok("V2 - Hotels list (new schema)");
    }
}

```

Artık şu iki istek farklı controller’lara gider:

```
GET /api/v1/hotels  → eski versiyon
GET /api/v2/hotels  → yeni versiyon

```

---

## 🧩 Alternatif Version Gönderim Yöntemleri

|Yöntem|Açıklama|Örnek|
|---|---|---|
|**URL segment**|En yaygın|`/api/v1/hotels`|
|**Query param**|Versiyonu query’de taşır|`/api/hotels?api-version=1.0`|
|**Header**|Versiyonu header’da taşır|`api-version: 1.0`|

---

## ⚙️ Swagger’da versiyonları göstermek

Ek paket:

```
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer

```

Ve Swagger’a entegre et:

```csharp
builder.Services.AddVersionedApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

```

Böylece Swagger UI’de:

```
v1
v2

```

olarak iki sekme çıkar.

---

## 📋 Mülakat notları

|Soru|Cevap|
|---|---|
|Neden API versioning gerekli?|Eski client’ların bozulmadan yeni sürümleri kullanabilmek için.|
|Hangi yöntemle versiyon verilir?|URL segment, query param veya header.|
|Default versiyon nedir?|Versiyon belirtilmezse hangi sürüm çalışsın diye belirlenir.|
|Swagger’da nasıl gösterilir?|`AddVersionedApiExplorer` kullanılır.|