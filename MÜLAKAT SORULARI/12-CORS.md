CORS, bir **güvenlik mekanizmasıdır.**

Tarayıcı, bir **domain (origin)** üzerinden gelen isteğin başka bir domain’e API çağrısı yapmasına izin vermez.

Örneğin 👇

|Frontend|Backend|Durum|
|---|---|---|
|`http://localhost:3000`|`http://localhost:5000`|❌ engellenir (farklı origin)|
|`http://localhost:5000`|`http://localhost:5000`|✅ izinli (aynı origin)|

Tarayıcı “farklı origin” algıladığında bir **Preflight Request (OPTIONS)** atar.

Bu istek, “Bu endpoint bana açık mı?” diye sorar.

Server `Access-Control-Allow-Origin` cevabı dönerse tarayıcı isteğe izin verir.

---

## 🧠 Neden önemli?

- Frontend (React, Angular, Vue) ile API genelde farklı portta veya domain’de çalışır.
    
- Eğer CORS açık değilse, API’ye istek atıldığında **browser tarafında hata alırsın**,
    
    ama **Postman’de çalışır.** (çünkü Postman tarayıcı güvenlik kısıtlamalarına tabi değildir.)
    

---

## ⚙️ Nasıl eklenir?

### 1️⃣ Program.cs → CORS policy tanımı:

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. CORS Policy tanımla
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend",
        policy =>
        {
            policy.WithOrigins("<http://localhost:3000>") // React/Angular dev portu
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});

```

### 2️⃣ Middleware olarak ekle:

```csharp
var app = builder.Build();

// 2. CORS aktif et (UseCors middleware'i ExceptionMiddleware'in ÜSTÜNDE olmalı)
app.UseCors("AllowFrontend");
app.UseMiddleware<ExceptionMiddleware>();
app.UseMiddleware<RequestLoggingMiddleware>();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapControllers();
app.Run();

```

---

## 🧩 Alternatif (Geçici - Geliştirme Aşaması)

Eğer şimdilik sadece test yapıyorsan:

```csharp
policy.AllowAnyOrigin()
      .AllowAnyHeader()
      .AllowAnyMethod();

```

> ⚠️ Uyarı: AllowAnyOrigin() sadece local geliştirmede kullanılmalıdır.
> 
> Üretim ortamında **yalnızca frontend domain’ini belirtmek gerekir.**

---

## 🧪 Nasıl test edilir?

Tarayıcı konsolunda bir hata görüyorsan şöyle yazar:

```
Access to fetch at '<http://localhost:5000/api/hotels>' from origin '<http://localhost:3000>' has been blocked by CORS policy.

```

Yukarıdaki policy eklenince hata gider.

Artık frontend API’ye güvenli şekilde erişebilir.

---

## 📋 Mülakat notları

|Soru|Cevap|
|---|---|
|CORS nedir?|Tarayıcıların farklı domain’lere yapılan istekleri engellemesini kontrol eden güvenlik mekanizmasıdır.|
|Neden Postman’de çalışıyor ama tarayıcıda çalışmıyor?|Postman CORS kontrolü yapmaz; tarayıcı yapar.|
|Preflight request nedir?|Tarayıcı, “OPTIONS” isteğiyle API’nin izin verip vermediğini sorar.|
|CORS policy nereye yazılır?|Middleware sırasına dikkat edilerek `Program.cs`’te `UseCors()` ile eklenir.|