
Bu konu, "Kod yazmak" ile "Yazılım Mimarisi tasarlamak" arasındaki eşiktir. Junior bir yazılımcı "Class'ın içinde `new` ile nesne oluştururum, çalışır" der. Senior bir mühendis ise "Asla `new` kullanma, Container yönetsin" der.

Önce, alt başlıklar olan yaşam döngülerine (Lifecycle) girmeden, DI'ın **felsefesini, neden var olduğunu ve IoC (Inversion of Control)** prensibiyle ilişkisini mühendislik derinliğinde inceleyelim.

---

### 1. Sorun: Sıkı Bağlılık (Tight Coupling)

DI'ı anlamak için önce "DI olmasaydı ne olurdu?" sorusuna bakmalıyız.

**Senaryo:** Bir e-ticaret siten var. `SiparisService` sınıfın, sipariş tamamlanınca müşteriye e-posta atıyor.

**DI Olmayan Kod (Kötü Tasarım):**

C#

```csharp
public class SiparisService
{
    private readonly EmailSender _emailSender;

    public SiparisService()
    {
        // HATA: Bağımlılığı kendin yarattın! (Tight Coupling)
        _emailSender = new EmailSender(); 
    }

    public void SiparisVer()
    {
        // ... sipariş kodları ...
        _emailSender.Gonder("Siparişiniz alındı");
    }
}
```

**Mühendislik Problemleri:**

1. **Değişim Zorluğu:** Yarın şirket "Artık Email değil, SMS atacağız" dedi. `SiparisService` kodunu açıp, `EmailSender` satırlarını silip `SmsSender` yazman gerekir. Ya bunu kullanan 100 tane servis varsa? Hepsini tek tek değiştirmelisin.
    
2. **Test Edilemezlik (Untestability):** Bu servise Unit Test yazmak istiyorsun. Testi çalıştırdığında `new EmailSender()` kodu çalışır ve **gerçekten e-posta atar.** Test yaparken müşterilere mail gitmesini istemeyiz. Araya "Sahte (Mock) Email Servisi" koyamazsın çünkü `new` kelimesiyle kodu kilitlemişsin.
    

---

### 2. Çözüm: Dependency Injection ve IoC Prensibi

IoC (Inversion of Control - Kontrolün Tersine Çevrilmesi):

Bu prensip der ki: "Bir sınıf, ihtiyaç duyduğu bağımlılıkları (Dependency) kendisi yaratmamalıdır (new). Bunun yerine, bu bağımlılıklar ona dışarıdan verilmelidir."

Buna **Hollywood Prensibi** de denir: _"Bizi arama, biz seni ararız."_

**DI Uygulanmış Kod (İyi Tasarım):**

C#

```csharp
public class SiparisService
{
    // Interface kullanıyoruz (Soyutlama)
    private readonly IBildirimService _bildirimService; 

    // Constructor Injection: "Bana dışarıdan bir bildirim servisi ver"
    public SiparisService(IBildirimService bildirimService)
    {
        _bildirimService = bildirimService;
    }

    public void SiparisVer()
    {
        // ...
        _bildirimService.Gonder("Sipariş alındı");
    }
}
```

**Mühendislik Kazancı:**

- `SiparisService` artık mail mi atılıyor, SMS mi atılıyor, dumanla mı haber veriliyor **bilmez.** Sadece `IBildirimService` arayüzünü bilir.
    
- Test yazarken içeriye sahte (Mock) bir servis verebilirsin.
    
- SMS'e geçmek için `SiparisService` koduna dokunmana gerek kalmaz.
    

---

### 3. DI Container (Orkestra Şefi)

Peki bu SiparisService'i kim oluşturacak? Constructor'ına kim parametre gönderecek?

İşte burada metinde geçen "DI Container" devreye girer.

ASP.NET Core'da bu işi yapan mekanizma `IServiceProvider`'dır. Uygulama ayağa kalkarken (`Program.cs`), biz Container'a tarifler veririz.

**Kayıt Aşaması (Program.cs):**

C#

```csharp
var builder = WebApplication.CreateBuilder(args);

// Tarif 1: Biri senden "IBildirimService" isterse, ona "EmailSender" ver.
builder.Services.AddScoped<IBildirimService, EmailSender>();

// Tarif 2: Biri senden "SiparisService" isterse, onu oluştur.
// (Container bakar: SiparisService ne istiyor? IBildirimService. 
//  O zaman önce EmailSender'ı oluşturur, sonra onu SiparisService'e verir.)
builder.Services.AddScoped<SiparisService>();
```

Bu mekanizma sayesinde devasa bir bağımlılık ağacı (Dependency Tree) otomatik olarak çözülür ve yönetilir.

---

### 4. Dependency Inversion Principle (DIP)

Bu konu SOLID prensiplerinin son harfidir (**D**). DI, bu prensibi uygulamanın yoludur.

> **Kural:** Yüksek seviyeli modüller (Sipariş Mantığı), düşük seviyeli modüllere (Mail Atma, Veritabanı Kaydı) doğrudan bağlı olmamalıdır. Her ikisi de **soyutlamalara (Interface)** bağlı olmalıdır.

Yukarıdaki örnekte `SiparisService` (Yüksek), `EmailSender` (Düşük) sınıfına değil, `IBildirimService` (Soyutlama) arayüzüne bağımlıdır. Mühendislik budur.

---

### 5. Enjeksiyon Türleri

1. **Constructor Injection (En Yaygını):** Bağımlılıklar kurucu metoddan istenir. Zorunlu bağımlılıklar için kullanılır (Örn: Veritabanı bağlantısı olmadan servis çalışmaz). %99 bunu kullanacaksın.
    
2. **Method Injection:** Bağımlılık sadece tek bir metoda lazımsa, o metodun parametresi olarak istenir (Minimal API'larda sık görülür).
    
3. **Property Injection:** Public bir property üzerinden verilir. (Zorunlu olmayan, opsiyonel bağımlılıklar için kullanılır ama .NET Core'da varsayılan olarak desteklenmez, 3. parti kütüphane gerekir).
    

---

