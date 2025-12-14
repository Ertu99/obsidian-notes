
Bir "Junior" ile "Senior" arasındaki en büyük farklardan biri, konfigürasyon yönetimidir. Junior, veritabanı şifresini kodun içine yazar (Hardcode). Senior ise bunu ortam değişkenlerinden (Environment Variables) okuyacak esnek bir yapı kurar.

Bu konuyu sadece "JSON dosyası okumak" olarak değil, modern DevOps dünyasının bel kemiği olan **Configuration Provider** mimarisi ve **Options Pattern** üzerinden inceleyeceğiz.

---

### 1. Felsefe: "Code" ve "Config" Ayrımı

12-Factor App (Modern bulut uygulamalarının anayasası) der ki: **"Konfigürasyonu koddan ayır."**

- **Kod:** Aynı kalır. (Derlenir, `.dll` olur).
    
- **Config:** Ortama göre değişir. (Local'de localhost DB, Prod'da Azure SQL DB).
    

Eğer veritabanı bağlantı cümleni (Connection String) kodun içine `const string` olarak yazarsan, Prod ortamına geçerken kodu değiştirip tekrar derlemen gerekir. Bu bir felakettir. ASP.NET Core, bu sorunu **Configuration API** ile çözer.

---

### 2. Katmanlı Mimari (Layered Providers)

ASP.NET Core'un konfigürasyon sistemi bir **Soğan (Onion)** gibidir. Ayarları tek bir yerden değil, sırayla birçok yerden okur ve **birbirinin üzerine yazar (Override).**

Standart bir .NET uygulamasında (Startup sınıfında veya Program.cs'te) yükleme sırası şöyledir:

1. `appsettings.json` (Temel ayarlar)
    
2. `appsettings.{Environment}.json` (Ortama özel ayar - Örn: Development)
    
3. **User Secrets** (Sadece geliştirme ortamındaki gizli şifreler)
    
4. **Environment Variables** (Docker/Kubernetes/Azure ortam değişkenleri)
    
5. **Command Line Arguments** (Terminalden girilen parametreler)
    

**Kritik Kural:** "Son gülen iyi güler." Listede en alttaki (son yüklenen), üsttekini ezer.

- `appsettings.json` -> `LogLevel: Warning`
    
- `Environment Variables` -> `LogLevel: Debug`
    
- **Sonuç:** Uygulama `Debug` modunda çalışır. Bu sayede kodu değiştirmeden sunucu ayarıyla uygulamanın davranışını değiştirebilirsin.
    

---

### 3. `appsettings.json` Yapısı

Bu dosya, ayarların saklandığı standart JSON dosyasıdır. Hiyerarşik bir yapıdadır.

JSON

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyDb;Trusted_Connection=True;"
  },
  "MailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "Port": 587
  }
}
```

Bu ayarlara kod içinden `IConfiguration` arayüzü ile erişiriz.

**Kötü Erişim (Magic String):**

C#

```csharp
// Tip güvenliği yok, yazım hatasına açık.
var port = _configuration["MailSettings:Port"]; 
```

---

### 4. Options Pattern (Mühendislik Çözümü)

Ayarları tek tek string olarak okumak yerine, onları bir **C# Sınıfına (Class)** bağlarız. Buna **Options Pattern** denir. Bu, kodun temiz (Clean Code) ve tip güvenli (Type Safe) olmasını sağlar.

1. **Sınıfı Oluştur:**
    
    C#
    
    ```csharp
    public class MailSettings
    {
        public string SmtpServer { get; set; }
        public int Port { get; set; }
    }
    ```
    
2. **Bind Et (Bağla) - Program.cs:**
    
    C#
    
    ```csharp
    // JSON'daki "MailSettings" bölümünü C# sınıfına dök.
    builder.Services.Configure<MailSettings>(builder.Configuration.GetSection("MailSettings"));
    ```
    
3. **Kullan (Dependency Injection):**
    
    C#
    
    ```csharp
    public class MailService
    {
        private readonly MailSettings _settings;
    
        // IOptions arayüzü ile enjekte edilir
        public MailService(IOptions<MailSettings> options)
        {
            _settings = options.Value;
        }
    }
    ```
    

---

### 5. Security: User Secrets vs Environment Variables

En büyük güvenlik açığı: `appsettings.json` içine gerçek şifreleri yazıp GitHub'a pushlamaktır.

- **Development Ortamı:** **User Secrets** kullanılır. Bu, proje klasörünün dışında, bilgisayarının gizli bir köşesinde (AppData/Roaming altında) tutulan şifreli olmayan bir JSON dosyasıdır. Git'e gitmez, sadece senin makinende çalışır.
    
- **Production Ortamı:** **Environment Variables** (Ortam Değişkenleri) veya **Azure Key Vault** kullanılır.
    

Asla, ama asla API Key veya DB şifresini `appsettings.json` içine yazıp commit atma.

---

### 6. İleri Seviye: Hot Reload (Canlı Güncelleme)

Uygulama çalışırken appsettings.json dosyasını değiştirirsen ne olur?

Standart IOptions<> (Singleton) kullanırsan hiçbir şey olmaz. Uygulamayı yeniden başlatman gerekir.

Ancak **`IOptionsSnapshot<>`** veya **`IOptionsMonitor<>`** kullanırsan, ASP.NET Core dosyadaki değişikliği algılar ve uygulamayı kapatıp açmadan yeni ayarları anında kodun içine yansıtır.

---

