## 🎯 Tanım:

**Middleware**, [ASP.NET](http://ASP.NET) Core uygulamasında her gelen HTTP isteği (request) ve cevabın (response) geçtiği **boru hattı (pipeline)** içindeki **ara katman (middle layer)** bileşenidir.

Yani:

> Middleware = “İstek (Request) sunucuya gelirken”
> 
> ve “Cevap (Response) kullanıcıya dönerken”
> 
> araya girip bir şeyler yapabilen kod parçalarıdır.

---

## 🧠 Basit düşünelim:

Bir HTTP isteği geldiğinde şu sırayla çalışır 👇

```
Kullanıcı → [Middleware 1] → [Middleware 2] → [Middleware 3] → Controller
Controller → [Middleware 3] → [Middleware 2] → [Middleware 1] → Kullanıcı

```

Her middleware, isteği bir sonraki middleware’e **aktarabilir** veya **durdurabilir**.

Aynı zamanda yanıt dönerken de işlem yapabilir (örneğin hata yakalama, loglama, caching vb.)

---

## 🔹 Middleware’in 2 temel görevi vardır:

1️⃣ **Request tarafında** çalışır → gelen isteği denetler, değiştirebilir.

2️⃣ **Response tarafında** çalışır → controller cevabı dönerken işleyebilir.

---

## ⚙️ Örnek olarak:

### 🔸 1. Authentication Middleware

→ “Kullanıcının token’ı var mı?”

→ Yoksa cevabı `401 Unauthorized` döner, pipeline durur.

### 🔸 2. Logging Middleware

→ “İstek hangi endpoint’e geldi, ne kadar sürdü?”

→ Log dosyasına yazar, sonra devam eder.

### 🔸 3. Exception Middleware

→ “Controller’da hata olursa yakala, logla, 500 döndür.”

### 🔸 4. Custom Middleware (senin yazdığın)

→ Kendi özel senaryolarını uygular (örneğin IP kontrolü, rate limit, tenant seçimi, header ekleme, vs.)

---

# 🧱 Middleware Nasıl Çalışır?

[ASP.NET](http://ASP.NET) Core, **pipeline** adını verdiği bir yapı oluşturur.

Sen bu pipeline’a middleware’leri sırayla eklersin:

```csharp
var app = builder.Build();

app.UseMiddleware<ExceptionMiddleware>();   // 1. bizim global error middleware’imiz
app.UseAuthentication();                    // 2. kimlik doğrulama
app.UseAuthorization();                     // 3. yetkilendirme
app.MapControllers();                       // 4. endpoint’ler (controller'lar)

app.Run();

```

📌 Burada `UseMiddleware<T>()` metodu o middleware sınıfını pipeline’a ekler.

---

# 💡 Middleware’in Temel Yapısı

Bir middleware genelde böyle görünür 👇

```csharp
public class ExampleMiddleware
{
    private readonly RequestDelegate _next;

    public ExampleMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // 1️⃣ İstek geldiğinde yapılacak işlemler
        Console.WriteLine($"İstek: {context.Request.Path}");

        // 2️⃣ Bir sonraki middleware’e gönder
        await _next(context);

        // 3️⃣ Cevap dönerken yapılacak işlemler
        Console.WriteLine($"Cevap: {context.Response.StatusCode}");
    }
}

```

Ve **Program.cs**’de:

```csharp
app.UseMiddleware<ExampleMiddleware>();

```

---

# 🚦 Middleware Akışı (Pipeline Diyagramı)

```
Request → [LoggingMiddleware]
         → [AuthenticationMiddleware]
         → [ExceptionMiddleware]
         → [Controller Action]
         ← [Response Dönüşü]

```

---

# ⚙️ Global Exception Middleware (projendeki 6. adım)

Senin projedeki “Global Exception Middleware” tam olarak şunu yapıyor:

> Controller içinde bir hata (exception) oluşursa uygulama çökmesin;
> 
> middleware o hatayı yakalasın, loglasın, ve kullanıcılara standart bir JSON hata cevabı dönsün.

Yani bu:

- “try-catch”’i tek tek her controller’a yazmak yerine
- tüm projeye **tek bir yerden** hata yönetimi eklemek anlamına gelir.

Buna **Global Error Handling Middleware** denir.

---

## 🔧 Örnek (senin projenle aynı mantıkta)

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionMiddleware> _logger;

    public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            // Bir sonraki middleware'e veya controller'a geç
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Beklenmeyen bir hata oluştu");
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";

            var error = new
            {
                message = ex.Message,
                detail = ex.InnerException?.Message
            };

            await context.Response.WriteAsJsonAsync(error);
        }
    }
}

```

Ve **Program.cs**’de:

```csharp
app.UseMiddleware<ExceptionMiddleware>();

```

Artık proje genelinde:

- Nerede hata olursa olsun, middleware yakalar.
- Uygulama çökmek yerine JSON döner:

```json
{
  "message": "Object reference not set to an instance of an object."
}

```

---

# 🧠 Mülakatlarda “Middleware yazmayı biliyor musun?” sorusu ne demek?

Mülakatçı aslında şunu anlamak ister:

> “Kendi middleware sınıfını yazabiliyor musun?
> 
> `RequestDelegate`, `HttpContext`, `InvokeAsync` yapısını biliyor musun?”

Yani evet ✅

**Projendeki Global Exception Middleware’i** kendin yazdığın için —

şu anda zaten “middleware yazmayı öğrenmiş” durumdasın.

Ama mülakatta genelde senden bunu **basit örnekle açıklaman** beklenir.

Mesela şöyle diyebilirsin 👇

> “[ASP.NET](http://ASP.NET) Core’da middleware, request ve response pipeline’ında çalışan ara katmanlardır.
> 
> Kendi middleware’imi `RequestDelegate` ile yazarım, `InvokeAsync` içinde isteği işlerim ve hata yönetimi, loglama gibi işleri merkezi hale getiririm.”

## 🧩 1. Middleware nereye aittir?

Middleware’ler:

- **HTTP pipeline** üzerinde çalışır,
- yani doğrudan **istek (request)** ve **cevap (response)** ile ilgilidir,
- bu da sadece **Web API katmanında** ([ASP.NET](http://ASP.NET) Core tarafında) bulunur.

🧠 Yani:

> Middleware’ler doğrudan [ASP.NET](http://ASP.NET) Core runtime’ı ile konuşur,
> 
> dolayısıyla **HotelBooking.Api** projesine eklenir.

---

## 📂 2. Katmanlar açısından bakarsak:

|Katman|Görev|Middleware eklenir mi?|
|---|---|---|
|**Domain**|Entity, business kuralları|❌ Hayır|
|**Application**|Servis, DTO, iş mantığı|❌ Hayır|
|**Infrastructure**|EF Core, veri erişimi|❌ Hayır|
|**API (Presentation)**|Controller, HTTP istekleri|✅ Evet (middleware burada çalışır)|

Yani ExceptionMiddleware gibi:

- LoggingMiddleware
- AuthenticationMiddleware
- ResponseWrapperMiddleware

gibi tüm HTTP tabanlı middleware’ler **API katmanında** yer almalıdır.

---

## ⚙️ 3. Dosya yapısı örneği (senin proje için)

```
HotelBooking.Api
│
├── Controllers
│   └── HotelsController.cs
│
├── Middlewares
│   └── ExceptionMiddleware.cs   ✅ burada olur
│
├── Program.cs                   ✅ burada UseMiddleware çağrılır
│
HotelBooking.Application
│   └── (Servisler, DTO’lar)
│
HotelBooking.Domain
│   └── (Entity’ler)
│
HotelBooking.Infrastructure
│   └── (DbContext, Repository)

```

---

## 🚀 4. Program.cs içinde nasıl devreye alınır?

API katmanında, `Program.cs` dosyasının **controller mapping’inden önce** eklenir 👇

```csharp
var app = builder.Build();

// Global Exception Middleware
app.UseMiddleware<ExceptionMiddleware>();

app.MapControllers();
app.Run();

```

📌 Sıra çok önemlidir:

- `app.UseMiddleware<ExceptionMiddleware>();` → en üstte olmalı (erken hata yakalasın)
- `app.MapControllers();` → en sonda olmalı (artık endpoint’lere gitmeye hazır)

---

## 🧠 5. Neden Application veya Infrastructure katmanına değil?

Çünkü:

- Middleware, **HTTP Request/Response** akışıyla ilgilidir.
- Application ve Infrastructure katmanları ise HTTP detaylarını **bilmemelidir** (temiz mimari prensibi).
- Domain veya Application içinde `HttpContext`, `Request`, `Response` kullanmak **katman bağımlılığını bozar**.

Yani:

> Middleware = dış dünya ile sistem arasındaki kapı.
> 
> Bu yüzden **sadece API katmanında** olmalıdır.

---

## ✅ 6. Özet (notlarına ekle)

|Soru|Cevap|
|---|---|
|Middleware neyle ilgilenir?|HTTP isteği ve cevabı ile|
|Hangi katmanda olmalı?|`HotelBooking.Api` (Web katmanı)|
|Neden Application değil?|Çünkü Application katmanı HTTP’den bağımsız olmalı|
|Nasıl devreye alınır?|`app.UseMiddleware<ExceptionMiddleware>();`|
|Dosya nereye eklenir?|`HotelBooking.Api/Middlewares/ExceptionMiddleware.cs`|

---

💬 **Kısaca:**

> Evet — middleware’ler (örneğin Global Exception Middleware),
> 
> **API katmanına** eklenir çünkü HTTP pipeline yalnızca o katmanda vardır.