

Middleware tüm trafiği (gelen giden her şeyi) etkilerken, Filtreler **cerrahi müdahale** yapar. Sadece belirli bir Controller'a veya belirli bir metoda özel kurallar koymanı sağlar.

Bu konuyu yazılım mühendisliğindeki **AOP (Aspect Oriented Programming - Cephe Yönelimli Programlama)** kavramı üzerinden inceleyeceğiz.

---

### 1. Filters Nedir? (Felsefe: Cross-Cutting Concerns)

Bir yazılımda "Loglama", "Yetkilendirme", "Hata Yakalama" gibi işler, projenin her yerine bulaşır. Her metodun içine `try-catch` yazmak veya her metodun başında `if (User.IsAdmin)` kontrolü yapmak kod tekrarıdır (DRY prensibine aykırıdır).

Bunlara **Cross-Cutting Concerns (Kesen İlgiler)** denir. Filtreler, bu tekrarlayan kodları metodun içinden söküp almanı ve metodun üzerine bir **Etiket (Attribute)** olarak yapıştırmanı sağlar.

- **Önce:**
    

C#

```csharp
public void DeleteUser() {
    // Kirlilik
    if (!User.IsAdmin) throw new UnauthorizedEx();
    Log("Silme başladı");
    
    _repo.Delete(); // Asıl iş (sadece 1 satır)
    
    Log("Silme bitti");
}
```

- **Sonra (Filtre ile):**
    

C#

```csharp
[Authorize(Roles = "Admin")] // Yetki Filtresi
[LogAction] // Özel Log Filtresi
public void DeleteUser() {
    _repo.Delete(); // Kod tertemiz
}
```

---

### 2. Middleware vs Filters (Mühendislik Kararı)

En çok karıştırılan konu budur: "Middleware de araya giriyor, Filter da araya giriyor. Hangisini kullanacağım?"

|**Özellik**|**Middleware**|**Filter**|
|---|---|---|
|**Konumu**|Controller'dan çok önce, en dıştaki boru hattındadır.|Controller'ın hemen dibindedir (MVC Context'ine hakimdir).|
|**Bilgi Düzeyi**|Düşüktür. Sadece HTTP Request'i (ham veriyi) görür. Hangi Action çalışacak bilmez.|Yüksektir. Hangi Controller, hangi Metot, parametreler nedir (`ModelState`) hepsini bilir.|
|**Kullanım**|Genel işler (HTTPS zorlama, statik dosyalar, global hata yakalama).|İnce işler (Model doğrulaması, belirli metoda özel cache, yetkilendirme).|

**Kural:** Eğer yapacağın iş `ModelState` (Action parametreleri) veya `ViewResult` ile ilgiliyse **Filter** kullan. Eğer tüm uygulamayı ilgilendiren genel bir işse **Middleware** kullan.

---

### 3. Filtrelerin Çalışma Sırası (The 5 Guardians)

ASP.NET Core'da filtreler rastgele çalışmaz. Çok katı bir hiyerarşisi vardır.

1. **Authorization Filters (İlk Güvenlik):** "İçeri girebilir misin?" En başta çalışır. Eğer red yerse, diğer filtreler çalışmaz.
    
2. **Resource Filters:** Model Binding (veri eşleşmesi) olmadan hemen önce çalışır. (Örn: Caching - Eğer cevap cache'te varsa, model oluşturmaya ve action'a gitmeye gerek yok, cache'ten dön).
    
3. **Action Filters (En Popüler):** Metot çalışmadan hemen önce (`OnActionExecuting`) ve hemen sonra (`OnActionExecuted`) çalışır. Parametreleri değiştirebilir, sonucu manipüle edebilirsin.
    
4. **Exception Filters:** Sadece o Controller/Action içinde bir hata (`throw ex`) olursa devreye girer.
    
5. **Result Filters:** HTML veya JSON cevabı oluşturulmadan hemen önce/sonra çalışır.
    

---

### 4. Custom Action Filter Yazmak (Kod Pratiği)

Kendi "Süre Ölçer" filtremizi yazalım ama bu sefer Attribute olarak.

C#

```csharp
// ActionFilterAttribute sınıfından miras alıyoruz
public class ExecutionTimeAttribute : ActionFilterAttribute
{
    private Stopwatch _watch;

    // Metot çalışmadan ÖNCE
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        _watch = Stopwatch.StartNew();
    }

    // Metot çalışıp bittikten SONRA
    public override void OnActionExecuted(ActionExecutedContext context)
    {
        _watch.Stop();
        // Cevabın Header'ına süreyi ekle
        context.HttpContext.Response.Headers.Add("X-Execution-Time", _watch.ElapsedMilliseconds.ToString());
    }
}
```

**Kullanımı:**

C#

```csharp
[ExecutionTime] // Artık bu metodun süresi ölçülecek
[HttpGet]
public IActionResult Get() { ... }
```

---

### 5. Dependency Injection Tuzağı (İleri Seviye)

Attribute'lar derleme zamanında (Compile Time) oluşturulan metadata'lardır. Bu yüzden içlerine **Constructor Injection** ile servis (Service) enjekte edemezsin.

**Hatalı:**

C#

```csharp
public class LogAttribute : ActionFilterAttribute {
    public LogAttribute(ILogger logger) { ... } // HATA! Attribute constructor'ı sadece sabit değer (string, int) alabilir.
}
```

**Çözüm:** `ServiceFilter` veya `TypeFilter` sarmalayıcılarını kullanmak.

**Doğrusu:**

C#

```csharp
// Controller üzerinde kullanımı:
[ServiceFilter(typeof(LogFilter))] // Servisi DI container'dan çözüp filtreye verir.
public IActionResult Get() { ... }
```

---

### 6. IActionFilter vs IAsyncActionFilter

Modern .NET dünyasında her şey asenkrondur (async/await). Eğer senkron (void OnActionExecuting) filtre kullanırsan thread bloklayabilirsin.

Mümkünse her zaman IAsyncActionFilter kullanmalısın.

C#

```csharp
public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
{
    // --- ÖNCE ---
    Console.WriteLine("Metot Başlıyor");

    // Action'ı çalıştır ve sonucu bekle (next)
    var resultContext = await next(); 

    // --- SONRA ---
    Console.WriteLine("Metot Bitti");
}
```

---

