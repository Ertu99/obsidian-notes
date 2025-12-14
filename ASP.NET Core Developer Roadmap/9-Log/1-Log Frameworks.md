
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

**🧒 6 Yaşındaki Çocuğa (Uçak Kara Kutusu Analojisi):** "Uçaklarda 'Kara Kutu' diye bir cihaz vardır. Pilotun her yaptığı şeyi kaydeder. Eskiden pilotlar deftere el yazısıyla 'Motor biraz ses çıkardı' yazardı (**Unstructured Log**). Kaza olduğunda bu yazıyı okumak çok zordu. Şimdiki modern kara kutular (**Serilog**), her şeyi bilgisayar diliyle kaydediyor: 'Motor: Sol, Hız: 500, Saat: 12.00' (**Structured Log**). Böylece bilgisayara 'Bana hızı 500 olan anları göster' dediğinde saniyesinde bulabiliyor. Ayrıca bu kara kutunun kabloları nereye bağlıysa bilgi oraya gider (**Sinks**). İster ekrana yazar, ister kuleye (Veritabanı) gönderir. Bir de havaalanında bavuluna taktıkları o barkod var ya (**Correlation ID**)? O bavul hangi uçağa binerse binsin, o barkod sayesinde bavulun nerede olduğunu hep bulabiliriz."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "Modern yazılım mimarisinde loglama, sadece hata yakalamak değil, sistemin röntgenini çekmek (Observability) demektir. Bu yüzden loglamayı **Structured Logging (Yapısal Loglama)** prensibiyle kurgulamak endüstri standardıdır. Klasik metin tabanlı loglar yerine, **Serilog** gibi kütüphaneler kullanarak logları JSON formatında (Key-Value) tutarız. Bu sayede Elasticsearch veya Seq gibi araçlarda, 'text search' yapmak yerine doğrudan property bazlı (`Where UserId = 5`) ışık hızında sorgular atabiliriz. Mikroservis mimarilerinde ise en kritik konu **Traceability (İzlenebilirlik)**tir. Bir isteğin sistemdeki tüm yolculuğunu tek bir ekranda görebilmek için, Gateway katmanında üretilen bir **Correlation ID**'nin tüm servisler boyunca taşınması ve `LogContext` ile her log satırına otomatik işlenmesi gerekir. Performans açısından ise, loglama işleminin ana thread'i (Main Thread) bloklamaması için I/O işlemlerinin **Asenkron (Async Sinks)** olarak arka planda yürütülmesi tercih edilir."

