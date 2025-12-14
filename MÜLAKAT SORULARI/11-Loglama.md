> Loglama, bir uygulamanın çalışma sürecinde gerçekleşen olayların (örneğin hatalar, istekler, uyarılar, bilgi mesajları) kayıt altına alınmasıdır.

Yani uygulama “ne zaman ne oldu” bilgisini dosyaya, veritabanına veya dış bir servise yazar.

Bu kayıtlar, yazılımın **takibini, hata tespitini** ve **performans analizini** kolaylaştırır.

---

## 🎯 **Neden Loglama Yapılır?**

|Amaç|Açıklama|
|---|---|
|🔍 **Hata takibi**|Hangi işlemin, hangi satırda, hangi sebeple hata verdiğini anlamak|
|⏱️ **Performans ölçümü**|İşlemler ne kadar sürüyor, nerede yavaşlama var|
|🧩 **Kullanıcı davranışı takibi**|Hangi endpoint’ler veya işlemler sık kullanılıyor|
|📜 **Denetim (audit)**|Kim, ne zaman, neyi değiştirdi|
|🛠️ **Bakım ve debugging**|Canlı sistemde sorun çıktığında geçmişi analiz etmek|

---

## 🧰 **.NET’te Loglama Nasıl Yapılır?**

[ASP.NET](http://ASP.NET) Core içinde gömülü bir **Logging altyapısı** vardır.

Kullanımı genelde şu şekildedir:

```csharp
private readonly ILogger<HomeController> _logger;

public HomeController(ILogger<HomeController> logger)
{
    _logger = logger;
}

public IActionResult Index()
{
    _logger.LogInformation("Home/Index called at {time}", DateTime.Now);
    return View();
}

```

### Log Seviyeleri:

|Seviye|Anlamı|Kullanım|
|---|---|---|
|`Trace`|En detaylı bilgi|Debug sırasında çok ayrıntılı kayıt|
|`Debug`|Geliştirme detayları|Sadece dev ortamında|
|`Information`|Genel bilgi mesajları|Normal akışta önemli olaylar|
|`Warning`|Uyarı|Olası problem veya beklenmeyen durum|
|`Error`|Hata|Hata oluştu ama uygulama çalışmaya devam ediyor|
|`Critical`|Ciddi hata|Uygulamanın durmasına neden olacak hata|

---

## ⚙️ **Loglama Nereye Yazılır?**

Loglar birçok hedefe yazılabilir:

- Konsol (`ConsoleLogger`)
- Dosya (örnek: `log.txt`)
- Veritabanı
- Cloud servisleri (Azure Application Insights, AWS CloudWatch, vs.)
- Üçüncü parti sistemler (örnek: **Serilog**, **NLog**, **Elasticsearch / Kibana**, **Seq**)

---

## 💡 **Serilog Örneği**

Serilog en popüler loglama kütüphanelerinden biridir.

Program.cs içinde genelde şöyle entegre edilir:

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("Logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();

builder.Host.UseSerilog();

```

> Böylece loglar hem konsola hem günlük dosyalarına yazılır.

---

## 🧾 **Özet (not olarak yaz)**

> Loglama, uygulamada gerçekleşen olayların kayıt altına alınması işlemidir.
> 
> **Amaç:** Hataları bulmak, performansı ölçmek, olay geçmişini izlemek.
> 
> **Seviyeler:** Trace, Debug, Information, Warning, Error, Critical
> 
> **Araçlar:** .NET built-in Logger, Serilog, NLog, Application Insights
> 
> **Kayıt Yeri:** Konsol, dosya, veritabanı, bulut servisi

# Serilog Nedir?

**Serilog**, .NET için **yapılandırılmış (structured) loglama** kütüphanesidir.

Düz metin yerine **anahtar–değer** (JSON benzeri) log yazar; böylece sorgulaması, filtrelemesi ve analizi çok kolay olur.

## Temel kavramlar

- **Structured logging:** `"Hotel created {HotelId} {City}"` → `HotelId=42, City="Istanbul"` olarak ayrı alanlara yazılır.
- **Sinks:** Logların nereye yazılacağını belirler (Console, File, Seq, Elasticsearch, SQL Server, App Insights…).
- **Enrichers:** Loglara otomatik ekstra alanlar ekler (ThreadId, MachineName, RequestId, UserName…).
- **MinimumLevel:** Log seviyeleri: `Verbose < Debug < Information < Warning < Error < Fatal`.

---

# Neden Serilog?

- **Analiz edilebilir**: JSON/log alanları ile “HotelId=42 iken hata verenler” gibi sorgular basitleşir.
- **Merkezi yönetim**: appsettings’ten seviyeleri/sink’leri değiştirebilirsin, kodu derlemeden.
- **Zengin ekosistem**: onlarca sink ve enricher; bulut/ELK/Seq ile entegre.

# Kullanım (ILogger)

Her yerde DI ile `ILogger<T>` kullan:

```csharp
public class HotelsController : ControllerBase
{
    private readonly ILogger<HotelsController> _logger;
    public HotelsController(ILogger<HotelsController> logger) => _logger = logger;

    [HttpPost]
    public async Task<IActionResult> Create(CreateHotelDto dto)
    {
        _logger.LogInformation("Create called for {City} - {Name}", dto.City, dto.Name);
        // ...
        _logger.LogInformation("Hotel created with {HotelId}", created.Id);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}

```

**Önemli:** String interpolasyon (`$"..."`) yerine **şablon + yer tutucu** (`{HotelId}`) kullan; bu, structured log alanı üretir.

---

# Global Exception Middleware ile birlikte

Middleware içinde Serilog kullan:

```csharp
catch (Exception ex)
{
    _logger.LogError(ex, "Unhandled exception at {Path} (TraceId: {TraceId})",
        context.Request.Path, context.TraceIdentifier);

    // 500 + standart JSON...
}

```

Bu sayede hem mesaj hem **stack trace** kaydedilir; `TraceId` ile istek–log eşleştirirsin.

---

# Enrichment (zenginleştirme)

- `Enrich.FromLogContext()` → `using (_logger.BeginScope(new { UserId = userId }))` ile scope alanlarını ekleyebilirsin.
- `Serilog.Enrichers.Environment` → `{MachineName}`, `{EnvironmentUserName}` otomatik gelir.
- İstek korelasyonu için `HttpContext.TraceIdentifier`’ı response’a koyup logla (ya da `WithCorrelationId` paketleri).

```csharp
using (_logger.BeginScope(new Dictionary<string, object> { ["UserId"] = userId }))
{
    _logger.LogInformation("Booking started"); // log'a UserId alanı eklenir
}

```

---

# İyi Pratikler

- **Structured** yaz (`{Field}`), concatenation/interpolation yapma.
    
- **Seviyeleri** doğru kullan:
    
    `Info` (iş akış olayı) / `Warning` (beklenmeyen ama tolere edilebilir) / `Error` (iş başarısız) / `Fatal` (uygulama çöküşü).
    
- **PII/Sırlar**: Şifre, token, kredi kartı gibi verileri asla loglama.
    
- **Filtreleme**: Gereksiz gürültüyü `Override` ile kıs; prod’da seviyeyi düşür.
    
- **Rolling file**: Dosyaları günlük döndür (`rollingInterval=Day`), elde tutulan sayıyı sınırlı tut.
    
- **Request logging**: `UseSerilogRequestLogging` ile süre, status, path otomatik loglansın.
    
- **Appsettings ile yönet**: Sinks/levels’i derlemeden değiştir.
    
- **ELK / Seq**: Orta–büyük projede mutlaka görselleştirme/arama aracı kullan (Seq en hızlı başlangıç).
    

---

# Sık Kullanılan Sinks (özet)

- **Console**: Lokal geliştirme.
- **File**: `logs/api-.log` (günlük dosya).
- **Seq**: Basit UI ile arama/filtre (dev/test ortamları için mükemmel).
- **Elasticsearch**: Kibana ile enterprise arama/analiz.
- **Application Insights**, **Datadog**, **Splunk**, **Graylog**…

---

# Hızlı Kontrol Listesi (projene uygula)

1. NuGet: `Serilog.AspNetCore`, `Serilog.Sinks.Console`, `Serilog.Sinks.File`.
2. `Program.cs` → `Log.Logger = new LoggerConfiguration()...` + `builder.Host.UseSerilog()`.
3. `appsettings.json` → `Serilog` bölümü (MinimumLevel, WriteTo, Enrich).
4. `app.UseSerilogRequestLogging()` ekle.
5. Controller/Service’lerde `ILogger<T>` ile **structured** log yaz.
6. ExceptionMiddleware’de `LogError(ex, "... {TraceId}", context.TraceIdentifier)`.

---

kısaca: **Serilog**, loglarını **alan bazlı** yazmanı sağlar; bu da **aranabilir, filtrelenebilir, görselleştirilebilir** loglar demek.

# **Serilog Neden Kullanılır? (Kısa ve Net)**

## ✅ 1. **[ASP.NET](http://ASP.NET) Core’un kendi log sistemi çok basit**

[ASP.NET](http://ASP.NET) Core’da varsayılan Logger vardır ama:

- Sadece konsola yazar
- Filtreleme kısıtlıdır
- JSON log yoktur
- Doküman/DB/Elastic’e gönderme yoktur
- Structured logging desteklemez

Bu yüzden büyük projelerde yetersiz kalır.

---

# 🌟 **Serilog’un Avantajları**

### 🟢 1) Structured Logging (En Önemlisi)

Serilog ile loglar _JSON_ gibi tutulur:

```json
{
  "Timestamp": "2025-11-18T10:00",
  "Message": "User {UserId} logged in",
  "UserId": 45
}

```

Bu yapı sayesinde:

- Arama yapmak kolaylaşır
- Hızlı analiz yapılır
- Loglar makine tarafından işlenebilir hale gelir

---

### 🟢 2) Birden Fazla Çıktıya (Sink) Yazabilir

Serilog logları şuralara gönderebilir:

- Console 🖥️
- File 📝
- SQL Server
- MongoDB
- Elasticsearch
- Seq
- Kibana
- Azure
- Email
- Telegram bile!

Varsayılan logger bunu yapamaz.

---

### 🟢 3) Çok güçlü filtreleme

Örneğin:

- “/api/login” isteklerini kaydet
- “Hangfire” loglarını alma
- Sadece Warning ve üzerini database’e kaydet
- Her endpoint’in süresini logla

Bu özelleştirmeler Serilog'da çok kolay.

---

### 🟢 4) Performanslı ve Asenkron

Serilog logları **asenkron** yazar → API yavaşlamaz.

Varsayılan logger ise genelde bloklayıcıdır.

---

### 🟢 5) Template-based logging

```csharp
Log.Information("Hotel {HotelId} created by {User}", id, userId);

```

Bu tarz structured message template’ler:

- ElasticSearch
- Kibana
- Seq

gibi araçlarla çok iyi çalışır.