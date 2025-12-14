
Microsoft'un kendi loglama kütüphanesi (`Microsoft.Extensions.Logging`) güzel bir soyutlamadır ama tek başına **yetersizdir**. Varsayılan olarak logları sadece konsola yazar ve yapısal (structured) özellikleri sınırlıdır.

Serilog ise bu soyutlamanın arkasına geçen, veriyi **JSON** olarak işleyen, **Enricher** (Zenginleştirici) ve **Destructuring** (Nesne Parçalama) gibi ileri mühendislik yeteneklerine sahip bir motordur.

Bu konuyu, Serilog'un en kritik 3 özelliği üzerinden, bir **Software Architect** derinliğinde inceleyelim.

---

### 1. The Magic Operator: Destructuring (`@`)

Serilog'u diğerlerinden ayıran ve mülakatlarda sorulan en kritik teknik özellik budur.

Normalde bir nesneyi loglamak istediğinde C# `.ToString()` metodunu çağırır.

C#

```cs
var user = new User { Id = 1, Name = "Ahmet" };
_logger.LogInformation("Kullanıcı: {User}", user);
```

- **Standart Çıktı:** `Kullanıcı: MyProject.Entities.User` (Sınıf adı yazar, işe yaramaz).
    

Serilog'da ise **`@` (Destructuring Operator)** kullanılır. Bu operatör Serilog'a şu emri verir: _"Bu nesneyi string'e çevirme, onu parçalarına ayır ve JSON nesnesi olarak sakla."_

C#

```cs
_logger.LogInformation("Kullanıcı: {@User}", user);
```

- **Serilog Çıktısı (JSON):**
    
    JSON
    
    ```js
    {
      "Message": "Kullanıcı: { Id: 1, Name: 'Ahmet' }",
      "User": { "Id": 1, "Name": "Ahmet" }
    }
    ```
    

**Mühendislik Gücü:** Artık Elasticsearch veya Seq üzerinde `User.Name == "Ahmet"` sorgusunu atabilirsin.

---

### 2. Request Logging Middleware (Gürültü Kirliliğini Önlemek)

ASP.NET Core'un varsayılan loglaması çok "gevezedir" (Noisy).

Bir HTTP isteği geldiğinde varsayılan logger şunları yazar:

1. `Request starting HTTP/1.1 GET ...`
    
2. `Routing matched endpoint ...`
    
3. `Executing endpoint ...`
    
4. `Executed endpoint ...`
    
5. `Request finished ...`
    

Tek bir istek için 5-10 satır log oluşur. Yüksek trafikte diskler anında dolar ve asıl hataları bulamazsın.

Serilog Çözümü: app.UseSerilogRequestLogging()

Bu middleware, ASP.NET Core'un o geveze loglarını kapatır ve yerine tek satırlık özet bir log atar.

- **Çıktı:** `HTTP GET /api/users responded 200 in 15.4 ms`
    
- Bu tek satırın içinde Payload, Status Code, Latency (Gecikme) ve Path bilgileri yapısal olarak bulunur.
    

**Mühendislik Notu:** Bu middleware'i `Program.cs` içinde **nerede** çağırdığın çok önemlidir. Statik dosyalardan (CSS/JS) sonra, MVC/Controller'dan önce çağırmalısın ki gereksiz resim dosyalarını loglamasın ama API isteklerini yakalasın.

---

### 3. Contextual Logging (Enrichment)

Loglara otomatik olarak bağlam (Context) ekleme sanatıdır.

Bir hata olduğunda sadece hatayı değil, o anki ortamı da bilmek istersin.

Serilog `Enrichers` paketiyle şunları otomatik ekler:

- **MachineName:** Hangi sunucuda oldu?
    
- **ThreadId:** Hangi thread çalışıyordu?
    
- **Environment:** Test mi, Prod mu?
    

Custom Context (LogContext):

Kodun belirli bir bloğu için geçici bilgi ekleyebilirsin.

C#

```cs
// Bu blok içindeki tüm loglara otomatik olarak "TransactionId" eklenir
using (LogContext.PushProperty("TransactionId", Guid.NewGuid()))
{
    _logger.LogInformation("Ödeme başladı");
    _paymentService.Process(); // Bu metodun içindeki loglarda da ID görünür!
    _logger.LogInformation("Ödeme bitti");
}
```

Bu sayede spagetti gibi birbirine girmiş asenkron logları ayrıştırabilirsin.

---

### 4. Sinks ve Konfigürasyon (Code vs JSON)

Serilog iki şekilde ayarlanabilir:

1. **Fluent API (Kod ile):** `new LoggerConfiguration().WriteTo.Console().CreateLogger();`
    
2. **appsettings.json (Konfigürasyon ile):** Önerilen yöntemdir. Log seviyesini veya hedefi (Sink) değiştirmek için kodu yeniden derlemen gerekmez.
    

**Popüler Sinks (Hedefler):**

- **File (Rolling File):** Dosyaya yazar ama dosya boyutu 10MB olunca yeni dosya açar (`log-20231027.txt`). Disk dolmasını engeller.
    
- **Seq:** Serilog'un yaratıcıları tarafından yapılan, logları izlemek için mükemmel bir arayüzdür. Development ortamında mutlaka kurup denemelisin.
    
- **Async Wrapper:** `WriteTo.Async(a => a.File(...))`
    
    - Bir önceki derste konuştuğumuz "I/O darboğazını" çözer. Loglamayı arka planda yapar, API'yi yavaşlatmaz.
        

---

### 5. Mühendislik Riski: Hassas Veri (PII)

Serilog'un `@` (Destructuring) operatörü çok güçlüdür dedik. Ama ya yanlışlıkla şunu yaparsan?

C#

```cs
var user = new User { Password = "123", CreditCard = "4444..." };
_logger.LogInformation("Kullanıcı giriş yaptı: {@User}", user);
```

**Felaket:** Kullanıcının şifresi ve kredi kartı açık açık log dosyasına (ve oradan Elasticsearch'e) yazılır. Bu, KVKK/GDPR gibi yasaların ihlalidir ve büyük güvenlik açığıdır.

**Çözüm:**

1. **ViewModel/DTO Kullan:** Loglarken asla Entity kullanma, sadece güvenli alanları olan DTO kullan.
    
2. **Destructuring Policy:** Serilog'a kural tanımlayabilirsin: _"Eğer `User` nesnesi loglanıyorsa, `Password` alanını görmezden gel."_
    

---

