JWT (JSON Web Token), taraflar arasında verilerin güvenli bir şekilde JSON formatında taşınmasını sağlayan açık bir standarttır (RFC 7519).

**Analoji:** Bir otele gittin (Login). Resepsiyonist kimliğine baktı ve sana bir **bileklik** taktı (JWT). Artık havuza, restorana veya odana girerken kimliğini göstermene gerek yok; bilekliğini gösteriyorsun. Otel personeli bilekliğe bakıp (Signature check) senin kim olduğunu ve hangi alanlara (Authorization) girebileceğini anlıyor.

---

### JWT'nin Anatomisi (3 Bölüm)

Bir JWT string'i `.` ile ayrılmış 3 bölümden oluşur: `aaaaa.bbbbb.ccccc`

#### 1. Header (Başlık)

Token'ın türünü (`JWT`) ve kullanılan şifreleme algoritmasını (`HS256`, `RS256`) belirtir.

JSON

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

#### 2. Payload (Yük / Veri)

Asıl verinin taşındığı kısımdır. Buradaki verilere **Claims** (İddialar) denir.

- **Standart Claims:** `sub` (Subject - Kullanıcı ID), `exp` (Expiration - Son kullanma tarihi), `iss` (Issuer - Kim üretti).
    
- **Custom Claims:** Senin eklediğin veriler (`role`: "Admin", `email`: "a@b.com").
    
- **UYARI:** Payload şifreli değildir! Sadece Base64 ile kodlanmıştır. Buraya **ASLA** şifre veya kredi kartı bilgisi koyma. Herkes çözüp okuyabilir.
    

JSON

```json
{
  "sub": "1234567890",
  "name": "Ahmet Yılmaz",
  "admin": true,
  "exp": 1700000000
}
```

#### 3. Signature (İmza) - En Kritik Kısım

Token'ın güvenliğini sağlayan yerdir. Sunucu; Header, Payload ve sadece kendisinin bildiği gizli bir anahtarı (**Secret Key**) birleştirip şifreler.

- Eğer bir hacker Payload'daki "admin: false" kısmını "admin: true" yaparsa, imza bozulur. Sunucu, gelen token'ın sahte olduğunu anlar.
    

---

### JWT Çalışma Mantığı (Akış)

1. **Login:** Kullanıcı `username` ve `password` ile API'ye istek atar (`POST /api/login`).
    
2. **Üretim (Generate):** Sunucu bilgileri doğrular. Doğruysa, gizli anahtarı (`SecretKey`) kullanarak bir JWT üretir.
    
3. **Teslimat:** Sunucu bu token'ı kullanıcıya döner.
    
4. **Saklama:** İstemci (Frontend/Mobil) bu token'ı saklar (LocalStorage veya Cookie).
    
5. **Kullanım:** İstemci, sonraki her istekte (örneğin "Siparişlerim") bu token'ı **Header** kısmında gönderir.
    
    - `Authorization: Bearer <token_burada>`
        
6. **Doğrulama (Validation):** Sunucu gelen token'ın imzasını ve süresini (`exp`) kontrol eder. Geçerliyse cevabı döner.
    

---

### C# ile JWT Uygulaması

Bir .NET Core projesinde JWT üretmek için genellikle `System.IdentityModel.Tokens.Jwt` paketi kullanılır.

#### 1. Token Üretme (Backend - Login Metodu)

C#

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

public string GenerateToken(User user)
{
    // 1. Secret Key (Bunu appsettings.json'dan okumalısın, burası sadece örnek)
    // Bu anahtar en az 16 karakter olmalı, yoksa hata alırsın.
    var secretKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("super_gizli_anahtarim_12345"));
    
    // 2. İmzalama algoritması
    var credentials = new SigningCredentials(secretKey, SecurityAlgorithms.HmacSha256);

    // 3. Claims (Token içine gömülecek veriler)
    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()), // Kullanıcı ID
        new Claim(JwtRegisteredClaimNames.Email, user.Email),       // Email
        new Claim("role", user.IsAdmin ? "Admin" : "User"),         // Rol (Custom Claim)
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()) // Token ID
    };

    // 4. Token Oluşturma
    var token = new JwtSecurityToken(
        issuer: "api.sitem.com",
        audience: "frontend.sitem.com",
        claims: claims,
        expires: DateTime.Now.AddMinutes(60), // 1 saat geçerli
        signingCredentials: credentials);

    // 5. String olarak döndür
    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

#### 2. Token Doğrulama (Program.cs Ayarı)

Sunucunun gelen token'ı otomatik tanıması için `Program.cs` içine şu ayarları eklersin:

C#

```csharp
// Program.cs

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true, // Süresi geçmiş mi? (exp kontrolü)
            ValidateIssuerSigningKey = true, // İmza doğru mu?
            
            ValidIssuer = "api.sitem.com",
            ValidAudience = "frontend.sitem.com",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("super_gizli_anahtarim_12345"))
        };
    });

// ... app build ...

app.UseAuthentication(); // Kimlik kontrolü (Sen kimsin?)
app.UseAuthorization();  // Yetki kontrolü (Girebilir misin?)
```

#### 3. Controller'da Kullanım (Authorization)

C#

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // Sadece token'ı olanlar girebilir
    [Authorize] 
    [HttpGet]
    public IActionResult GetMyOrders()
    {
        // Token içindeki ID'yi okuma (User.Identity üzerinden)
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Ok($"Kullanıcı ID: {userId} için siparişler getiriliyor...");
    }

    // Sadece 'Admin' rolü olanlar girebilir
    [Authorize(Roles = "Admin")]
    [HttpDelete("{id}")]
    public IActionResult DeleteOrder(int id)
    {
        return Ok("Sipariş silindi.");
    }
}
```

---

### Backend Developer İçin Kritik İpuçları

1. **Secret Key Güvenliği:** Yukarıdaki kodda `super_gizli_anahtarim...` string'ini koda gömmek (Hardcode) büyük hatadır. Bunu `appsettings.json`'da veya daha iyisi sunucunun **Environment Variables** (Ortam Değişkenleri) kısmında saklamalısın. Eğer bu anahtarı kaptırırsan, hackerlar kendi kendilerine "Admin" token'ı üretip sistemine girebilirler.
    
2. **HTTPS Zorunluluğu:** JWT token'ı her istekte açık bir şekilde (Header içinde) gider. Eğer siten `HTTPS` (SSL) değilse, ağdaki biri (Man-in-the-Middle) bu token'ı havada kapabilir ve senin yerine kullanabilir.
    
3. **Token Boyutu:** Payload kısmına ne kadar çok veri (`Claim`) eklersen token o kadar şişer. Her istekte bu verinin sunucuya gidip geleceğini unutma. Sadece gerekli verileri (ID, Email, Rol) koy. Kullanıcının biyografisini koyma.
    
4. **Logout Sorunu:** JWT stateless olduğu için, sunucu "Bu token'ı iptal et" diyemez (Token süresi bitene kadar geçerlidir).
    
    - _Çözüm:_ Token sürelerini kısa tutmak (Access Token: 15 dk) ve yanında uzun süreli bir **Refresh Token** kullanmaktır. Kullanıcı "Çıkış Yap" dediğinde client tarafındaki token silinir.