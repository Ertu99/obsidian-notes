Şimdi ise dışarıdan hiçbir kütüphane almadan, .NET Core'un kendi motorunun içinde gelen, **"Built-in" (Gömülü)** arka plan mekanizmasına, **Native Background Services** konusuna giriyoruz.

Çoğu geliştirici bunu basit bir "Timer" sanar. Ama bir Mimar olarak sen bunu; **Singleton Yaşam Döngüsü**, **Scope Yönetimi (En Kritik Kısım)** ve **Graceful Shutdown** prensipleriyle yönetmelisin.

Attığın metin `IHostedService` arayüzünden bahsetmiş. Biz bu konuyu, **Dependency Injection tuzakları** ve **Worker Service** mimarisi üzerinden, tek seferde ve eksiksiz inceleyelim.

---

### 1. Anatomi: `IHostedService` vs `BackgroundService`

.NET Core'da arka plan işi yazmanın iki yolu vardır. Farkı bilmek önemlidir.

#### A. `IHostedService` (Ham Arayüz)

- İki metodu vardır: `StartAsync` ve `StopAsync`.
    
- **Zorluk:** `StartAsync` metodu, uygulama ayağa kalkarken çalışır. Eğer burada `while(true)` döngüsü kurarsan, uygulama **asla ayağa kalkamaz.** (Boot işlemini bloklarsın).
    
- **Kullanım:** Sadece uygulama başlarken bir kere çalışacak işler için (Örn: Veritabanı Migration'ı tetiklemek).
    

#### B. `BackgroundService` (Modern Sınıf)

- `IHostedService`'i implemente eden abstract bir sınıftır.
    
- **Kolaylık:** `ExecuteAsync` adında bir metot verir. Bu metot arka planda **ayrı bir Task** olarak çalıştırılır.
    
- **Güvenlik:** Burada sonsuz döngü kurabilirsin, ana uygulamayı (Web API) bloklamaz.
    
- **Mühendislik Standartı:** Arka plan işi yazacaksan her zaman `BackgroundService` sınıfından miras al.
    

---

### 2. En Büyük Tuzak: DI Scope Problemi (Singleton vs Scoped)

Bu konu mülakatlarda seni Senior yapar veya eler.

- **Kural 1:** `BackgroundService` (Hosted Service), uygulama yaşam döngüsü boyunca **Singleton** olarak kaydedilir. (Uygulama başladığında doğar, kapanana kadar yaşar).
    
- **Kural 2:** `DbContext` (Veritabanı bağlantısı), varsayılan olarak **Scoped** (İstek başına) yaşam döngüsüne sahiptir.
    

**Hata (Junior Yaklaşımı):**

C#

```cs
public class MyWorker : BackgroundService
{
    private readonly AppDbContext _context; // HATA!

    // Singleton içine Scoped servisi Constructor'dan enjekte edemezsin!
    public MyWorker(AppDbContext context) 
    {
        _context = context;
    }
}
```

**Sonuç:** Uygulama çalışırken hata fırlatır: _"Cannot consume scoped service 'AppDbContext' from singleton 'MyWorker'."_

Çözüm (Service Scope Factory):

Kendi Scope'unu kendin yaratmalısın.

C#

```cs
public class MyWorker : BackgroundService
{
    private readonly IServiceProvider _provider;

    public MyWorker(IServiceProvider provider)
    {
        _provider = provider;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // 1. Manuel Scope Oluştur
            using (var scope = _provider.CreateScope())
            {
                // 2. Servisi Scope içinden al
                var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
                
                // 3. İşini yap
                var users = await context.Users.ToListAsync();
            } // 4. Scope burada Dispose olur (Bağlantı kapanır, RAM temizlenir)
            
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

---

### 3. Zamanlama: `Task.Delay` vs `PeriodicTimer`

Eskiden döngü içinde `Thread.Sleep` veya `Task.Delay` kullanırdık. .NET 6 ile gelen **`PeriodicTimer`** çok daha hassas ve modern bir yöntemdir.

C#

```cs
var timer = new PeriodicTimer(TimeSpan.FromMinutes(1));

while (await timer.WaitForNextTickAsync(stoppingToken))
{
    // Dakikada bir çalışacak kod
}
```

- **Farkı:** `PeriodicTimer`, işlemin ne kadar sürdüğünden bağımsız olarak (süre kayması olmadan) tik tak çalışır. Ayrıca `CancellationToken` ile tam uyumludur.
    

---

### 4. İşletim Sistemi Entegrasyonu: Windows Service & Systemd

Yazdığın bu BackgroundService, varsayılan olarak Kestrel (Web Sunucusu) ile birlikte yaşar.

Peki ya sen hiç Web API istemiyorsan? Sadece arka planda çalışan bir "Daemon" istiyorsan?

.NET Core sana **"Worker Service"** şablonunu sunar. Bu proje türü, yazdığın kodu işletim sistemi servisine dönüştürür.

1. **Windows:** `Install-Package Microsoft.Extensions.Hosting.WindowsServices`
    
    - `builder.Host.UseWindowsService();`
        
    - Kodun artık Windows Hizmetler (services.msc) altında çalışabilir.
        
2. **Linux:** `Install-Package Microsoft.Extensions.Hosting.Systemd`
    
    - `builder.Host.UseSystemd();`
        
    - Kodun Linux Daemon olarak çalışır.
        

---

### 5. Graceful Shutdown (Nazikçe Kapanma)

Bir güncelleme yapacaksın ve sunucuyu kapatıyorsun. O sırada Worker tam veritabanına yazıyordu.

İşletim sistemi kapanma sinyali (SIGTERM) gönderdiğinde, ExecuteAsync metodundaki stoppingToken tetiklenir (IsCancellationRequested = true olur).

**Mühendislik Kuralı:** Kodunda her `await` işlemine (`Task.Delay`, `SaveChangesAsync`, `HttpClient.GetAsync`) mutlaka `stoppingToken` parametresini geçirmelisin.

- **Yanlış:** `await Task.Delay(5000);` (Kapanma sinyali gelse bile 5 saniye bekler, işletim sistemi zorla öldürür).
    
- **Doğru:** `await Task.Delay(5000, stoppingToken);` (Sinyal geldiği an bekleme iptal olur, `TaskCanceledException` fırlatır ve döngüden hemen çıkılır).
    

---

### 6. Queue Processing (Kuyruk Tüketimi)

Native Background Service'in en yaygın kullanım alanı **RabbitMQ** veya **Azure Service Bus** dinlemektir.

- **Producer (Web API):** Mesajı kuyruğa atar.
    
- **Consumer (BackgroundService):**
    
    C#
    
    ```cs
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _rabbitMqConsumer.Received += (model, ea) => 
        {
            // Mesaj geldiğinde çalış
            ProcessMessage(ea.Body);
        };
    
        // Sonsuza kadar bekle (Thread'i öldürme)
        await Task.Delay(Timeout.Infinite, stoppingToken);
    }
    ```
    

---

**🧒 6 Yaşındaki Çocuğa (Gece Bekçisi Analojisi):** "Gündüzleri dükkan açıkken (Web API) müşterilerle ilgilenen tezgahtarlar vardır. Akşam dükkan kapanınca herkes evine gider ama dükkanda birinin kalması gerekir. İşte **Background Service**, dükkanın **Gece Bekçisi** gibidir. Müşterilerle konuşmaz, satış yapmaz. Sadece arka planda sessizce dolaşır, ışıkları kontrol eder, yerleri süpürür. Bu bekçinin çok önemli bir kuralı vardır: Dükkanın kasasını açamaz (**Scoped Service**). Çünkü kasa anahtarı sadece gündüz tezgahtarlarında durur. Eğer bekçi kasadan bir şey almak isterse, müdürden özel izinle bir anahtar ister, işini halleder ve anahtarı hemen geri verir (**Creating Scope**). Bir de bekçinin kulağı hep kapıdadır. Eğer müdür gelip 'Dükkanı tamamen kapatıyoruz' derse (**Cancellation Token**), bekçi elindeki süpürgeyi olduğu yere bırakmaz; güzelce yerine koyar, ışığı kapatır ve öyle çıkar (**Graceful Shutdown**)."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** ".NET Core ekosisteminde arka plan işlemleri için harici kütüphanelere bağımlı kalmadan, yerleşik (Native) çözümler kullanılması modern mimaride önceliklidir. Burada iki temel yapı taşı kullanılır:

- **BackgroundService:** `IHostedService` arayüzünü implemente eden, uzun soluklu (Long-running) işlemler için optimize edilmiş bir soyutlamadır.
    
- **Worker Service:** Uygulamanın bir Web API olmadan, doğrudan Linux Daemon (Systemd) veya Windows Service olarak çalışabilmesini sağlayan proje şablonudur. Mimari tasarımda dikkat edilmesi gereken en kritik nokta **Dependency Injection ve Scope Yönetimi**dir. BackgroundService 'Singleton' olarak yaşadığı için, 'Scoped' servisleri (örn: DbContext) doğrudan constructor'dan alamaz. Bunun yerine `IServiceScopeFactory` kullanılarak manuel bir scope oluşturulmalı ve iş bitince dispose edilmelidir. Ayrıca, sistemin kararlılığı (Reliability) için **Graceful Shutdown** mekanizması hayati önem taşır. `ExecuteAsync` metoduna gelen `CancellationToken`, tüm asenkron işlemlere (`Task.Delay`, I/O calls) geçirilerek, uygulama kapanırken işlemlerin yarım kalması engellenir."