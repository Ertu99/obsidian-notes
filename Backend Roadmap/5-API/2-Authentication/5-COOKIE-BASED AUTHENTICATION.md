Authentication başlığının son ve en köklü konusu olan **Cookie-Based Authentication**'a geldik.

JWT ve Token sistemleri "Modern Web"in (SPA, Mobil, Microservice) yıldızı olsa da, geleneksel web uygulamaları (MVC, Razor Pages, PHP, Django) hala %90 oranında Cookie kullanır. Bankacılık sistemleri ve devlet siteleri genellikle bu yöntemi tercih eder. Nedenini ve nasıl çalıştığını derinlemesine inceleyelim.

---

### Nedir Bu Cookie? (Devlet Dairesi Analojisi)

JWT için "Bileklik" veya "Kimlik Kartı" demiştik. Cookie-Based Authentication ise **"Devlet Dairesindeki Randevu Defteri"** gibidir.

1. **Giriş:** Memura kimliğini gösterirsin (Login).
    
2. **Kayıt:** Memur önündeki **kocaman deftere** (Session Store / Server Memory) adını yazar ve sana sıra numarası **"105"** olan küçücük bir kağıt verir.
    
3. **İşlem:** Sen başka bir odaya gittiğinde, kimliğini göstermezsin. Sadece "105" numaralı kağıdı gösterirsin.
    
4. **Kontrol:** Diğer memur defteri açar, "105 numara... Hımm, evet Ahmet Bey'miş, yetkisi var" der ve işlemi yapar.
    

- **Fark:** JWT'de tüm bilgi senin elindeki kartta yazardı. Cookie'de ise bilgi **sunucuda** (Defterde) yazar, sende sadece o bilginin **Referans ID**'si (Kağıt) vardır. Bu yüzden buna **Stateful** (Durum Tutan) yöntem denir.
    

---

### Nasıl Çalışır? (Teknik Akış)

Tarayıcılar (Chrome, Safari), Cookie'leri yönetmek için doğuştan yeteneklidir. Senin kod yazmana gerek kalmadan bazı işleri otomatik yaparlar.

1. **Login İsteği:** Kullanıcı `username` ve `password` gönderir.
    
2. **Session Oluşturma:** Sunucu bilgileri doğrular. Sunucu hafızasında (veya Redis/SQL'de) bir oturum (Session) oluşturur ve buna bir ID verir (örn: `abc-123`).
    
3. Set-Cookie: Sunucu cevabı dönerken, HTTP Header'ına şunu ekler:
    
    Set-Cookie: SessionId=abc-123; HttpOnly; Secure
    
4. **Saklama:** Tarayıcı bu Header'ı görür ve "Tamam, bu site için bu cookie'yi saklamalıyım" der.
    
5. **Otomatik Gönderim:** Kullanıcı o site içinde nereye tıklarsa tıklasın, tarayıcı **otomatik olarak** her isteğin içine bu Cookie'yi ekler. Senin JavaScript ile `fetch` yazarken header eklemene gerek yoktur.
    

---

### Güvenlik Riskleri ve Çözümleri (CSRF & XSS)

Cookie kullanıyorsan, güvenlik ayarlarını (Flags) çok iyi bilmelisin. Yoksa siten süzgece döner.

#### 1. XSS (Cross-Site Scripting) ve `HttpOnly`

Eğer bir hacker sitene zararlı bir JavaScript kodu sızdırırsa (`<script>alert(document.cookie)</script>`), kullanıcının oturum ID'sini çalıp hesabını ele geçirebilir.

- **Çözüm:** `HttpOnly` bayrağı.
    
- Cookie oluştururken `HttpOnly=true` dersen, tarayıcıdaki JavaScript (React/Vue kodun dahil) bu cookie'yi **OKUYAMAZ**. Sadece tarayıcı arka planda sunucuya gönderir. Bu, XSS saldırılarına karşı muazzam bir kalkandır. (JWT'de LocalStorage kullanırsan bu koruma yoktur).
    

#### 2. CSRF (Cross-Site Request Forgery)

Cookie'nin en büyük baş belasıdır. Tarayıcının "Her isteğe cookie'yi otomatik ekleme" özelliği burada bize ihanet eder.

- **Senaryo:** Banka hesabına giriş yaptın (Cookie tarayıcıda). Yan sekmede "Hediye Kazandın!" diye bir siteye girdin. O sitede gizli bir buton var ve arkada bankana "Hesabıma para gönder" isteği atıyor.
    
- **Saldırı:** Tarayıcı, istek bankaya gittiği için, senin banka Cookie'ni de otomatik olarak o isteğe ekler. Banka sunucusu isteğin senden geldiğini sanır ve parayı gönderir.
    
- **Çözüm:** `SameSite` ve `Anti-Forgery Token`.
    
    - `SameSite=Strict` veya `Lax`: Tarayıcıya "Eğer istek başka bir siteden geliyorsa Cookie'yi gönderme" emri verir.
        

---

### C# / ASP.NET Core ile Cookie Auth Uygulaması

JWT'de `AddJwtBearer` kullanmıştık. Burada `AddCookie` kullanacağız. ASP.NET Core'un Identity sistemi arka planda bunu kullanır ama biz manuel (saf) halini yazalım.

#### 1. Program.cs Konfigürasyonu

C#

```csharp
var builder = WebApplication.CreateBuilder(args);

// Cookie Authentication Servisini ekle
builder.Services.AddAuthentication("MyCookieAuth") // Şema adı
    .AddCookie("MyCookieAuth", options =>
    {
        options.Cookie.Name = "SiteminOturumCerezi"; // Cookie adı
        options.LoginPath = "/Account/Login"; // Yetkisiz giriş yapanı buraya at
        options.ExpireTimeSpan = TimeSpan.FromMinutes(30); // 30 dk sonra oturum düşsün
        
        // Güvenlik Ayarları (Çok Önemli!)
        options.Cookie.HttpOnly = true; // JS okuyamasın (XSS koruması)
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // Sadece HTTPS (Man-in-middle koruması)
        options.Cookie.SameSite = SameSiteMode.Strict; // CSRF koruması
    });

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

#### 2. Login (Cookie Oluşturma)

Kullanıcı adı şifre doğruysa, tarayıcıya "Seni tanıdım, al bu bileti" deme kısmı (`HttpContext.SignInAsync`).

C#

```csharp
public class AccountController : Controller
{
    [HttpPost]
    public async Task<IActionResult> Login(string username, string password)
    {
        // 1. Veritabanı kontrolü (Simülasyon)
        if (username != "ahmet" || password != "123")
            return Unauthorized("Hatalı giriş");

        // 2. Kimlik Kartı Bilgilerini Hazırla (Claims)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, username),
            new Claim(ClaimTypes.Role, "Admin"),
            new Claim("FavoriRenk", "Mavi")
        };

        var claimsIdentity = new ClaimsIdentity(claims, "MyCookieAuth");
        var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);

        // 3. Cookie'yi oluştur ve tarayıcıya gönder (Şifreleyip gönderir)
        // ASP.NET Core burada veriyi şifreler (Data Protection), client okuyamaz.
        await HttpContext.SignInAsync("MyCookieAuth", claimsPrincipal);

        return Ok("Giriş Başarılı");
    }
}
```

#### 3. Logout (Cookie Silme)

C#

```csharp
    [HttpPost]
    public async Task<IActionResult> Logout()
    {
        // Tarayıcıya "O cookie'yi sil" emri gönderir (Set-Cookie: ...; Expires=GecmisTarih)
        await HttpContext.SignOutAsync("MyCookieAuth");
        return Ok("Çıkış yapıldı");
    }
```

---

### Backend Developer Karar Anı: Cookie mi Token mı?

Bu tabloyu başucuna asabilirsin:

|**Özellik**|**Cookie (Session)**|**Token (JWT)**|
|---|---|---|
|**Depolama**|Sunucu tarafında (Stateful).|Client tarafında (Stateless).|
|**İstemci**|Tarayıcı (Browser) için mükemmeldir.|Mobil App ve SPA için mükemmeldir.|
|**Güvenlik**|CSRF koruması şart. XSS'e karşı HttpOnly ile daha güvenli.|XSS'e karşı hassas (LocalStorage çalınabilir).|
|**Ölçeklenebilirlik**|Zor (Redis gibi ortak Session deposu gerekir).|Kolay (Sunucular birbirinden bağımsızdır).|
|**Domain**|Sadece kendi domaininde çalışır (Cross-Domain zordur).|Her domainde çalışır (API için ideal).|

**Özet:**

- Eğer **MVC** projesi yapıyorsan veya sadece Web üzerinden hizmet veriyorsan -> **Cookie** kullan.
    
- Eğer **Mobil Uygulama** ve **React/Angular** (SPA) yapıyorsan -> **JWT (Token)** kullan.