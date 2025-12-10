
Şimdi Unit Test'in laboratuvar ortamından çıkıp, kodun gerçek dünyada nasıl davrandığını test ettiğimiz yere, **Integration Testing (Entegrasyon Testleri)** ve onun kalbi olan **WebApplicationFactory**'ye geliyoruz.

Çoğu yazılımcı entegrasyon testi yapmak için; projeyi çalıştırır (F5), Postman'i açar ve manuel istek atar. Bu sürdürülebilir değildir. **WebApplicationFactory (WAF)**, senin uygulamanı (Program.cs) alır, arka planda **bellek üzerinde (In-Memory)** gerçek bir sunucu gibi ayağa kaldırır ve sana bu sunucuyla konuşman için özel bir `HttpClient` verir.

Bu konuyu; **Program.cs Bootstrapping**, **Service Replacement (Bağımlılık Değiştirme)** ve **TestContainers** ile gerçek veritabanı kullanımı üzerinden eksiksiz inceleyelim.

---

### 1. Felsefe: "Grey Box" Testing

Unit Test "White Box" idi (Kodun içini biliyorduk). E2E Test "Black Box" idi (Kodun içini bilmiyorduk). **WebApplicationFactory** ise **Grey Box**'tır.

- Uygulamaya dışarıdan HTTP isteği atarsın (Black Box gibi).
    
- Ama istersen içerideki Dependency Injection kutusuna elini sokup bazı parçaları değiştirebilirsin (White Box gibi).
    

---

### 2. Mimari: Nasıl Çalışır?

`Microsoft.AspNetCore.Mvc.Testing` paketi ile gelir. Test projesinde `IClassFixture<WebApplicationFactory<Program>>` arayüzünü implemente edersin.

1. **Bootstrapping:** WAF, senin gerçek projenin `Program.cs` dosyasını bulur ve `Main` metodunu çalıştırır. Yani Middleware'ler, Filtreler, Route'lar **gerçekte nasılsa testte de öyledir.**
    
2. **TestServer:** Kestrel (gerçek sunucu) yerine, istekleri doğrudan işleyen hafif bir `TestServer` başlatır. Ağ trafiği (Network Latency) yoktur, çok hızlıdır.
    
3. **HttpClient:** `factory.CreateClient()` dediğinde, bu TestServer'a bağlı özel bir istemci alırsın.
    

C#

```cs
public class ProductIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ProductIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetProducts_ShouldReturnOk()
    {
        var response = await _client.GetAsync("/api/products");
        response.EnsureSuccessStatusCode(); // 200 OK mi?
    }
}
```

---

### 3. Mühendislik Gücü: Service Replacement (Bağımlılık Değiştirme)

Entegrasyon testinin en kritik noktası burasıdır. Gerçek veritabanını kullanmak istiyorsun ama **Gerçek Ödeme Sistemini (Iyzico/Stripe)** kullanmak istemiyorsun (Her testte kredi kartından para çekilmesin).

WAF, uygulama ayağa kalkmadan hemen önce `ConfigureWebHost` metoduyla araya girip DI konteynerini manipüle etmene izin verir.

C#

```cs
var factory = new WebApplicationFactory<Program>()
    .WithWebHostBuilder(builder =>
    {
        builder.ConfigureServices(services =>
        {
            // 1. Gerçek Ödeme Servisini Bul ve Sil
            var descriptor = services.SingleOrDefault(d => d.ServiceType == typeof(IPaymentService));
            services.Remove(descriptor);

            // 2. Sahtesini (Mock) Ekle
            services.AddScoped<IPaymentService, FakePaymentService>();
        });
    });
```

Artık test çalışırken `/api/checkout` endpoint'ine istek attığında, arkada senin sahte servisin çalışır.

---

### 4. Veritabanı Stratejisi: In-Memory vs TestContainers

Entegrasyon testlerinde en büyük tartışma veritabanıdır.

1. **EF Core In-Memory DB (Eski Yöntem - Önerilmez):**
    
    - Hızlıdır ama SQL Server gibi davranmaz. (Örn: SQL'de Constraint hatası veren kayıt, In-Memory'de hatasız kaydedilebilir). Test geçer ama Canlı patlar.
        
2. **TestContainers (Modern Standart - Önerilen):**
    
    - Test başladığında **Docker** üzerinde gerçek bir "MS SQL Server" konteyneri ayağa kaldırır.
        
    - Connection String'i dinamik olarak WAF'a enjekte eder.
        
    - Test bitince konteyneri imha eder.
        
    - **Sonuç:** %100 Production uyumlu test.
        

---

### 5. Authentication Bypass (Sahte Giriş)

API'lerin çoğu `[Authorize]` ile korunur. Her testten önce `/api/login` endpoint'ine istek atıp Token almak ve onu Header'a eklemek büyük zaman kaybıdır.

**Mühendislik Çözümü: TestAuthHandler** ASP.NET Core'un Authentication yapısını "Hack"leriz.

- `TestAuthHandler` adında özel bir handler yazarız.
    
- Bu handler, gelen her isteği "Başarılı" kabul eder ve isteğe sahte bir "Admin User Claim" ekler.
    
- WAF konfigürasyonunda bu handler'ı kaydederiz.
    

Böylece testlerde `client.GetAsync("/admin/dashboard")` dediğinde, sistem seni giriş yapmış Admin sanar.

---

### 6. AppSettings Manipülasyonu

Test ortamında farklı ayarlar kullanmak isteyebilirsin (Örn: Cache süresi 1 saniye olsun). `appsettings.Test.json` dosyası oluşturup WAF'a bunu kullanmasını söyleyebilirsin.

C#

```cs
builder.ConfigureAppConfiguration((context, config) =>
{
    config.AddJsonFile("appsettings.Test.json");
});
```



---

Şimdi Entegrasyon Testlerinin "Kutsal Kasesi"ne, **"Fake Database" yalanından kurtulup**, test ortamında %100 gerçek altyapı kullanmamızı sağlayan devrime: **Testcontainers**'a geliyoruz.

Eskiden test yaparken "EF Core In-Memory Database" kullanırdık.

- **Sorun:** In-Memory veritabanı Foreign Key (İlişki) kısıtlamalarını kontrol etmez. SQL'e özgü (JSON Column, Spatial Data) özellikleri desteklemez.
    
- **Sonuç:** Testlerin geçer ("Yeşil" yanar) ama kodun Canlıya çıktığında SQL hatası verip patlar. Buna **"False Positive"** denir ve bir mühendisin en büyük kabusudur.
    

Testcontainers; Docker API'sini kullanarak test başladığında **gerçek** bir konteyner (SQL, Redis, RabbitMQ) ayağa kaldıran ve test bitince yok eden bir kütüphanedir.

Bu konuyu; **Lifecycle (Yaşam Döngüsü)**, **Port Mapping (Dinamik Portlar)** ve **CI/CD Entegrasyonu** üzerinden derinlemesine inceleyelim.

---

### 1. Felsefe: "Disposable Infrastructure" (Kullan-At Altyapı)

Testcontainers'ın mantığı şudur: Test ortamı kalıcı olmamalıdır.

- Test başlarken: _"Bana sıfır kilometre bir SQL Server 2019 ver."_
    
- Test sırasında: _"İçine tabloları oluştur, veriyi bas, testi yap."_
    
- Test bitince: _"Sunucuyu imha et."_
    

Bu sayede her test (veya test sınıfı), tertemiz ve izole bir veritabanında çalışır.

---

### 2. Kurulum ve `IAsyncLifetime`

xUnit ile Testcontainers kullanırken, konteynerin ayağa kalkmasını beklememiz gerekir. Constructor (Yapıcı Metot) asenkron (`await`) çalışamaz. Bu yüzden xUnit'in **`IAsyncLifetime`** arayüzünü kullanırız.

C#

```cs
public class ProductTests : IAsyncLifetime
{
    // 1. Konteyner Tanımı (Henüz çalışmıyor)
    private readonly MsSqlContainer _dbContainer = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    // 2. Test Başlamadan Önce (Initialize)
    public async Task InitializeAsync()
    {
        // Docker'a "Konteyneri Başlat" emri gider. (2-3 saniye sürer)
        await _dbContainer.StartAsync();
        
        // KONTEYNER HAZIR! 
        // Connection String'i alıp WebApplicationFactory'ye vermeliyiz.
        var connectionString = _dbContainer.GetConnectionString();
    }

    // 3. Test Bitince (Dispose)
    public async Task DisposeAsync()
    {
        // Konteyneri öldür ve temizle.
        await _dbContainer.DisposeAsync();
    }
}
```

---

### 3. Kritik Mühendislik Detayı: Dinamik Port Yönetimi

Junior geliştiriciler genelde şunu sorar: _"Connection string'i neden kodla alıyoruz? `appsettings.test.json` içine `localhost,1433` yazsak olmaz mı?"_

**Cevap: ASLA OLMAZ.**

Testcontainers, konteyneri ayağa kaldırırken **Random Port Binding** yapar.

- Konteynerin içindeki port: **1433** (Standart SQL).
    
- Bilgisayarındaki (Host) port: **55123** (Rastgele).
    

Neden?

Çünkü aynı anda (Paralel) 5 farklı test sınıfı çalışabilir. Hepsi 1433'ü kullanmaya çalışırsa "Port Conflict" hatası alırsın. Testcontainers her teste ayrı bir port vererek %100 izolasyon sağlar.

Bu yüzden Connection String'i **Runtime (Çalışma Anı)** sırasında `_dbContainer.GetConnectionString()` ile öğrenip, WAF (WebApplicationFactory) konfigürasyonunu ezmemiz gerekir.

C#

```cs
// WAF Konfigürasyonu
var factory = new WebApplicationFactory<Program>()
    .WithWebHostBuilder(builder =>
    {
        builder.ConfigureServices(services =>
        {
            // Eski DbContext ayarını sil
            // Yeni Connection String ile DbContext'i ekle
            services.AddDbContext<AppDbContext>(options => 
                options.UseSqlServer(_dbContainer.GetConnectionString())); 
        });
    });
```

---

### 4. Sadece SQL Değil: The Ecosystem

Testcontainers sadece veritabanı için değildir. Uygulamanın bağımlı olduğu her şeyi Dockerize edebilirsin:

- **Redis:** Cache testleri için.
    
- **RabbitMQ:** Message Broker testleri için.
    
- **LocalStack:** AWS servislerini (S3, SQS, DynamoDB) taklit etmek için.
    
- **MailHog:** SMTP sunucusu taklidi (Mail atıldı mı kontrolü) için.
    

Bu, E2E testlerin "Gerçek Dünya"ya en yakın simülasyonudur.

---

### 5. Performans Maliyeti ve "Shared Container"

Her test metodu ([Fact]) için yeni bir Docker konteyneri kaldırmak çok yavaştır (Her test +3 saniye gecikir).

100 testin varsa test süresi 5 dakika uzar.

Mühendislik Çözümü: Collection Fixture

Tüm test paketi için Tek Bir Konteyner kaldırılır.

1. Testler başlar -> Konteyner kalkar.
    
2. Tüm 100 test bu konteyneri kullanır.
    
3. Testler biter -> Konteyner iner.
    

_Not: Bu durumda "State Pollution" (Veri Kirliliği) riski geri gelir. Her testten önce tabloları `TRUNCATE` eden bir ara katman (örneğin `Respawn`) kullanmak şarttır._

---

### 6. CI/CD Pipeline (GitHub Actions / Azure DevOps)

Testcontainers, yerel bilgisayarında (Docker Desktop) harika çalışır. Peki kodu GitHub'a attığında (CI) ne olur?

GitHub Actions sunucuları (Runner) zaten Ubuntu üzerindedir. Testcontainers'ın çalışması için "Docker-in-Docker" (DinD) veya "Docker Socket Binding" gerekir.

Modern CI araçlarının çoğu bunu destekler. Sadece pipeline konfigürasyonunda Docker servisini aktif etmen yeterlidir. Ekstra bir SQL Server kurmana gerek kalmaz, kodun kendi veritabanını kendi yanında getirir.

---
