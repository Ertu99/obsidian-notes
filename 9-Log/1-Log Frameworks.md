
Bir Junior yazılımcı için loglama, Console.WriteLine("Hata oluştu") yazmaktan ibarettir.

Bir Mimar içinse Loglama; sistemin nabzını tutan, hata anında (Post-Mortem) otopsi yapmayı sağlayan ve dağıtık sistemlerde (Microservices) iz sürmeyi (Tracing) mümkün kılan hayati bir organdır.

Bu konuyu, özellikle .NET ekosisteminin standardı haline gelen **Serilog** ve modern dünyanın vazgeçilmezi **Structured Logging (Yapısal Loglama)** kavramları üzerinden derinlemesine inceleyelim.

---

### 1. Mimari: Abstraction vs Implementation (Soyutlama ve Uygulama)

.NET Core, loglama konusunda harika bir mimariyle gelir: Microsoft.Extensions.Logging.

Bu, loglamanın arayüzüdür (Interface).

- **Facade (Ön Yüz):** Sen kodun içinde sadece `ILogger<T>` arayüzünü kullanırsın.
    
- **Provider (Sağlayıcı):** Arka planda logu nereye ve nasıl atacağını belirleyen kütüphanedir (Serilog, NLog, Log4Net).
    

**Mühendislik Faydası:** Kodunun içine `Serilog` bağımlılığı eklemezsin. Sadece Microsoft'un arayüzüne bağımlısın. Yarın "NLog'a geçiyoruz" dersen, kodundaki `logger.LogInformation(...)` satırlarını değiştirmene gerek kalmaz. Sadece `Program.cs` ayarını değiştirirsin.

---

### 2. Devrim: Structured Logging (Yapısal Loglama)

Eskiden loglar düz metin (String) olarak tutulurdu. Bu, günümüzün Büyük Veri dünyasında bir felakettir.

**Eski Usül (Unstructured):**

C#

```cs
var userId = 5;
var action = "Login";
// Metin birleştirme
logger.LogInformation("Kullanıcı " + userId + " sisteme " + action + " yaptı.");
```

- **Disktek Hali:** `"Kullanıcı 5 sisteme Login yaptı."`
    
- **Sorun:** "ID'si 5 olan kullanıcının tüm hareketlerini bul" dediğinde, veritabanında `LIKE '%Kullanıcı 5%'` gibi çok yavaş bir metin araması (Regex) yapman gerekir.
    

**Modern Usül (Structured - Serilog):**

C#

```cs
// Message Template (Mesaj Şablonu)
logger.LogInformation("Kullanıcı {UserId} sisteme {Action} yaptı.", userId, action);
```

- **Diskteki Hali (JSON):**
    
    JSON
    
    ```js
    {
      "Timestamp": "2023-10-27T10:00:00",
      "Message": "Kullanıcı 5 sisteme Login yaptı.",
      "Properties": {
         "UserId": 5,
         "Action": "Login"
      }
    }
    ```
    
- **Fayda:** Elasticsearch veya Seq gibi araçlarda `SELECT * FROM Logs WHERE UserId = 5` diyerek, metin aramadan, doğrudan alan (Field) üzerinden sorgu atabilirsin. Işık hızındadır.
    

---

### 3. Serilog ve "Sink" Mimarisi

Serilog, .NET dünyasının en popüler kütüphanesidir çünkü **"Sink" (Lavabo/Gider)** mantığıyla çalışır. Log bir su gibidir, onu nereye akıtacağına sen karar verirsin.

- **Console Sink:** Terminale yazar (Development ortamı için).
    
- **File Sink:** Dosyaya yazar (`log.txt`).
    
- **Elasticsearch Sink:** Logları analiz için Elastic'e atar.
    
- **Seq Sink:** Yapısal logları izlemek için harika bir araçtır.
    
- **MSSQL Sink:** Veritabanına yazar (Genelde önerilmez, DB'yi yorar).
    

Enrichment (Zenginleştirme):

Serilog, loglara otomatik olarak ekstra bilgi ekleyebilir.

.Enrich.WithMachineName() dersen, logu atan sunucunun adını her satıra ekler. Hangi sunucuda hata olduğunu anında bulursun.

---

### 4. Correlation ID (İz Sürme)

Microservices veya dağıtık sistemlerde (Distributed Systems) bir hata olduğunda en büyük sorun şudur: **"Bu hata nerede başladı?"**

- Kullanıcı "Sipariş Ver"e bastı (API Gateway).
    
- Gateway -> Sipariş Servisine gitti.
    
- Sipariş Servisi -> Stok Servisine gitti.
    
- Stok Servisi -> Hata verdi!
    

Loglara baktığında Stok Servisinde bir hata görürsün ama bunu hangi kullanıcının, hangi sipariş isteğinin tetiklediğini bilemezsin.

**Çözüm: Correlation ID**

1. İstek sisteme ilk girdiği an (Gateway), ona benzersiz bir GUID (`X-Correlation-ID`) atanır.
    
2. Bu ID, servisler arası HTTP header'larında taşınır.
    
3. Serilog, `LogContext` kullanarak bu ID'yi **tüm log kayıtlarına** otomatik ekler.
    
4. Sen `WHERE CorrelationId = 'abc-123'` dediğinde, o isteğin tüm servislerdeki yolculuğunu (Story) tek ekranda görürsün.
    

---

### 5. Performans: Asenkron Loglama

Loglama işlemi I/O Bound (Diske veya Ağa yazma) bir işlemdir.

Eğer her LogInformation dediğinde sistemin diske yazmasını beklersen, API cevap süresi uzar.

**Mühendislik Kuralı:** Loglama asla ana thread'i (Main Thread) bloklamamalıdır.

- **Serilog.Sinks.Async:** Bu kütüphane ile loglar önce bellekteki bir kuyruğa (Queue) atılır.
    
- Arka plandaki bir işçi (Worker Thread) bunları sırayla diske yazar.
    
- API cevabı milisaniyede döner, log arkadan gelir. (Tek risk: Uygulama çökerse kuyruktaki son birkaç log kaybolabilir).
    

---

### 6. Log Seviyeleri (Log Levels)

Hangi logun önemli olduğuna karar vermek, disk alanını ve gürültüyü yönetmek için kritiktir.

1. **Trace / Verbose:** En detaylı, her şey. (Sadece geliştirirken açılır).
    
2. **Debug:** Hata ayıklama bilgileri. (Canlı ortamda kapalıdır).
    
3. **Information:** İş akışı. "Sipariş alındı", "Mail atıldı". (Genel akış).
    
4. **Warning:** Hata değil ama riskli. "Bakiye az kaldı", "İşlem 3 saniye sürdü (yavaş)".
    
5. **Error:** Beklenmedik hata. `try-catch` bloklarında yakalananlar.
    
6. **Critical / Fatal:** Sistem çöktü. Uygulama kapanıyor.
    

**Production Ayarı:** Genelde `Information` veya `Warning` seviyesidir. `Debug` açarsan diskler saatler içinde dolar.

---

