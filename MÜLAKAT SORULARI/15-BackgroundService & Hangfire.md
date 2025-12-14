Bu konuyla birlikte API’nin “arka planda” nasıl işler çalıştırdığını,

örneğin:

- belirli aralıklarla veri temizleme,
- e-posta gönderimi,
- rapor oluşturma,
- otomatik hatırlatma gibi **zamanlanmış işler** nasıl yapılır — bunları öğreneceğiz.

---

# 🔹 1. Arka plan işleri neden ayrı çalışır?

HTTP istekleri **stateless**tir → kullanıcı isteği biter bitmez thread serbest bırakılır.

Ama bazı işler **istek bittikten sonra da devam etmelidir.**

Örneğin:

- Kullanıcı “Rapor Oluştur” dedi → rapor 10 dakika sürüyor.
- API hemen 200 OK döner ama rapor arka planda oluşturulur.

İşte burada “arka plan servisleri” devreye girer.

---

# 🔹 2. .NET’te BackgroundService temeli

[ASP.NET](http://ASP.NET) Core’da background işlemler **IHostedService** arabirimiyle yapılır.

`BackgroundService`, bu arabirimin basit bir implementasyonudur.

# 3. Arka plan servisleri nasıl çalışır?

- `BackgroundService` uygulama başlarken bir **Task** başlatır.
- Uygulama kapanana kadar bu task çalışmaya devam eder.
- `CancellationToken` durdurulduğunda (örneğin uygulama kapanırken) gracefully sonlanır.

Her BackgroundService, `IHostedService` olarak kayıtlıdır → [ASP.NET](http://ASP.NET) Core host tarafından yönetilir.

---

# 🔹 4. Gerçek dünyada — Hangfire

`BackgroundService` güzel ama:

- Kodla zamanlama yapmak zor (Timer veya Delay).
- Dashboard yok.
- Job geçmişi tutulmaz.

Bu yüzden büyük projelerde genelde **Hangfire** kullanılır.

Hangfire:

> .NET için açık kaynaklı, background job framework’üdür.
> 
> SQL Server’da job’ları saklar ve bir dashboard sunar.

# 9. Hangfire Job Türleri

|Tür|Açıklama|
|---|---|
|**Fire-and-forget**|Anında bir kere çalışır.|
|**Delayed**|Belirli süre sonra çalışır.|
|**Recurring**|Belirli periyotlarla tekrarlar (cron).|
|**Continuation**|Başka bir job bitince çalışır.|

---

# 🔹 10. Mülakat notları

| Soru                                    | Cevap                                                                                                              |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| IHostedService nedir?                   | [ASP.NET](http://ASP.NET) Core içinde uzun süreli arka plan işlemleri başlatmak için kullanılan temel arabirimdir. |
| BackgroundService ne yapar?             | IHostedService’in abstract implementasyonu; arka planda sürekli çalışan döngüsel işleri yönetir.                   |
| Hangfire ne işe yarar?                  | Arka plan job’larını yönetir, zamanlar, saklar ve dashboard sağlar.                                                |
| Hangfire job tipleri?                   | Fire-and-forget, delayed, recurring, continuation.                                                                 |
| Hangfire veritabanı olarak ne kullanır? | SQL Server (veya Redis, PostgreSQL gibi alternatifler).                                                            |