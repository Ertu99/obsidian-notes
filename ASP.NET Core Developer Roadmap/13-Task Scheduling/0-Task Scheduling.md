
Bir web uygulamasında (ASP.NET Core), kullanıcı "Kayıt Ol" butonuna bastığında, sen arka planda "Hoş geldin maili at", "Rapor oluştur", "Resmi sıkıştır" gibi işlemleri o an (senkron olarak) yaparsan; kullanıcıyı 10 saniye bekletirsin. Kullanıcı sıkılır ve sekmeyi kapatır.

Bu yüzden **Task Scheduling (Görev Zamanlama)** ve **Background Jobs (Arka Plan İşleri)** hayati önem taşır. Bu konuyu; **Built-in (Gömülü)** çözümlerden **Distributed (Dağıtık)** sistemlere, **Persistence (Kalıcılık)** ve **Graceful Shutdown** mühendisliğine kadar tek seferde ve eksiksiz inceliyoruz.

---

### 1. Felsefe: Fire-and-Forget vs Recurring vs Delayed

Arka plan işleri temelde 3 kategoriye ayrılır:

1. **Fire-and-Forget (Ateşle ve Unut):** "Şu maili sıraya al, ne zaman müsait olursan gönder." Kullanıcı cevabı beklemez.
    
2. **Delayed (Gecikmeli):** "Bu işi şu an yapma, 15 dakika sonra yap." (Örn: Sepette ürün unutan kullanıcıya hatırlatma at).
    
3. **Recurring (Tekrarlayan/CRON):** "Her gece 03:00'te günlük raporu oluştur."
    

---

### 2. Seviye 1: The Native Way (`IHostedService` & `BackgroundService`)

ASP.NET Core'un içinde gömülü gelen en temel yapıdır. Ekstra kütüphane gerektirmez.

- **Nasıl Çalışır:** Uygulama ayağa kalkarken (`Program.cs`), senin yazdığın `Worker` sınıfı da bir Thread üzerinde ayağa kalkar ve uygulama kapanana kadar sonsuz döngüde çalışır.
    

C#

```cs
public class LogWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Log temizleniyor...");
            await Task.Delay(TimeSpan.FromHours(1), stoppingToken);
        }
    }
}
// Program.cs
builder.Services.AddHostedService<LogWorker>();
```

- **Avantajı:** Bedavadır, kurulum gerektirmez.
    
- **Risk (Volatility):** Bellek (RAM) üzerinde çalışır. Eğer sunucu resetlenirse (Deploy sırasında veya hata alıp çökerse), o an kuyrukta bekleyen tüm işler **KAYBOLUR.** (Mail gitmez).
    
- **Kullanım:** Sadece önemsiz işler (Cache temizleme, Log dosyasını döndürme) için kullanılır.
    

---

### 3. Seviye 2: The Enterprise Way (Hangfire & Quartz.NET)

Kritik işler (Para çekme, Mail atma) için "Kaybolma" riskini alamayız. Bize **Persistence (Kalıcılık)** lazım. İşte burada devler devreye girer.

#### A. Hangfire (Sektör Standardı)

.NET dünyasının en popüler kütüphanesidir.

- **Persistence:** İşleri RAM'de değil, **Veritabanında (SQL Server / Redis)** saklar. Sunucu patlasa bile, tekrar açıldığında "Nerede kalmıştık?" der ve devam eder.
    
- **Dashboard:** Muazzam bir arayüzü vardır. Hangi iş hata verdi, hangisi sırada, tek tıkla görebilir ve "Requeue" (Tekrar Dene) diyebilirsin.
    
- **Retry Mechanism:** Bir iş hata verirse (SMTP sunucusu kapalıysa), Hangfire onu otomatik olarak 10 kere daha dener (Exponential Backoff ile).
    

C#

```cs
// Fire-and-Forget
BackgroundJob.Enqueue(() => mailService.SendWelcomeEmail(userId));

// Recurring
RecurringJob.AddOrUpdate("rapor-olustur", () => reportService.CreateDailyReport(), Cron.Daily);
```

#### B. Quartz.NET (Eski Toprak)

Java dünyasından port edilmiştir.

- **Gücü:** Çok karmaşık zamanlama senaryolarını (Örn: "Her ayın son Cuma günü hariç, hafta içi sabahları çalış") destekler.
    
- **Zayıflığı:** Kurulumu ve kullanımı Hangfire'a göre çok daha zordur. Dashboard'u gömülü gelmez.
    

#### C. Coravel (Modern & Basit)

Laravel'in "Task Scheduling" yapısını sevenler için yapılmıştır. Söz dizimi çok temizdir (`scheduler.Schedule<MyJob>().EveryMinute()`). Ancak Hangfire kadar güçlü bir Persistence ve Dashboard sunmaz. Hafif işler için idealdir.

---

### 4. Mimari Karar: In-Process vs Out-of-Process

Bir Mimar olarak vermen gereken en kritik karar şudur: **Bu işler nerede çalışacak?**

1. **In-Process (Aynı Süreçte):**
    
    - API projesi ile Hangfire Server **aynı projede** çalışır.
        
    - _Sorun:_ Eğer Rapor Oluşturma işlemi %100 CPU harcarsa, API yavaşlar ve kullanıcılar siteye giremez.
        
2. **Out-of-Process (Ayrı Worker Service):**
    
    - **Proje A (Web API):** Sadece işi kuyruğa ekler (Enqueue).
        
    - **Proje B (Worker Service):** Sadece kuyruğu dinler ve işi yapar.
        
    - **Veritabanı:** İkisi de aynı Hangfire veritabanına bağlıdır.
        
    - _Fayda:_ Rapor oluşturulurken Web API etkilenmez. Worker Service'i ayrı sunucuda ölçekleyebilirsin.
        

---

### 5. Mühendislik Detayı 1: Graceful Shutdown

Uygulamana yeni bir versiyon deploy ediyorsun. O sırada arka planda kritik bir "Para Transferi" işlemi yapılıyor.

Sunucu aniden kapanırsa ne olur? İşlem yarım kalır, veri tutarsızlaşır.

`IHostedService` ve Hangfire, **Cancellation Token** mekanizmasını kullanır.

- İşletim sistemi "Kapan" sinyali (SIGTERM) gönderdiğinde, ASP.NET Core `stoppingToken.IsCancellationRequested = true` yapar.
    
- Senin kodun bunu kontrol etmeli ve "Tamam, şu anki döngüyü bitireyim, yenisine başlamayayım" diyerek nazikçe kapanmalıdır.
    

### 6. Mühendislik Detayı 2: Overlapping Execution (Çakışan Çalışma)

**Senaryo:** "Her dakika çalış" dediğin bir işin var. Ama işin bitmesi 3 dakika sürüyor.

- 09:00 -> İş A başladı.
    
- 09:01 -> İş B başladı (İş A hala bitmedi!).
    
- 09:02 -> İş C başladı (A ve B hala çalışıyor!).
    
- **Sonuç:** Sunucu kilitlenir.
    

**Çözüm:**

- **Hangfire:** `[DisableConcurrentExecution]` attribute'u ile "Önceki bitmeden yenisi başlama" diyebilirsin.
    
- **Quartz:** `DisallowConcurrentExecution` kullanırsın.
    

---

### 7. Mühendislik Detayı 3: Idempotency (Yine)

Task Scheduling dünyasında "En az bir kere çalışma" (At-least-once delivery) garantisi vardır. Ama "Sadece bir kere" garantisi zordur.

Hangfire bir işi tamamladı sanıp hata alabilir ve tekrar deneyebilir.

Eğer senin kodun "Kredi Kartından Çek" ise, müşteriden 2 kere para çekebilirsin.

**Kural:** Arka plan işleri **Idempotent** yazılmalıdır.

- _"Para çek"_ DEĞİL, _"ID: 123 olan siparişin ödemesi alınmadıysa çek"_ şeklinde kontrol koyulmalıdır.
    

---
