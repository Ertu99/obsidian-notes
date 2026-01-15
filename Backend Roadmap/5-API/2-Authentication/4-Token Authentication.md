JWT ve OAuth konularında "Token" kelimesini çok kullandık ama bu konuyu genel bir şemsiye başlık olarak toparlamak ve "Session Based" (Oturum Bazlı) sistemlerle farkını netleştirmek şart. Çünkü mülakatlarda sana **"Neden Session değil de Token kullanıyoruz?"** diye soracaklar.

### Token Authentication Nedir?

Kullanıcının kimlik bilgilerini (Kullanıcı Adı/Şifre) verip, karşılığında dijital bir **geçiş bileti (Token)** aldığı sistemdir. Kullanıcı artık şifresini değil, bu bileti göstererek işlem yapar.

**Analoji:** Bir gece kulübüne gidiyorsun. Kapıdaki görevliye kimliğini gösteriyorsun (Login). Görevli koluna damga basıyor veya bileklik takıyor (Token). Artık içeride her içecek alışında kimlik göstermiyorsun, kolunu gösteriyorsun.

---

### En Büyük Rakip: Session Based Authentication

Token mantığını anlamak için, eskiden ne yaptığımızı (Session) bilmen gerekir.

#### 1. Session Based (Eski / Stateful)

- **Mantık:** Kullanıcı giriş yapınca sunucu, kendi hafızasında (RAM) veya veritabanında "Ahmet şu an içeride" diye bir kayıt (Session ID) oluşturur. Bu ID'yi kullanıcıya **Cookie** olarak verir.
    
- **Sorun:** Sunucu durum tutar (Stateful).
    
    - Eğer uygulaman çok büyür ve 2. sunucuyu kurarsan (Load Balancing), Ahmet 1. sunucuda giriş yapmıştır ama 2. sunucunun bundan haberi yoktur. Ahmet 2. sunucuya düştüğünde sistem onu tanımaz. (Redis ile çözülür ama maliyetlidir).
        
- **Mobil Uyumu:** Mobil uygulamalar (Android/iOS) Cookie yönetimi konusunda tarayıcılar kadar yetenekli değildir, zorluk çıkarır.
    

#### 2. Token Based (Modern / Stateless)

- **Mantık:** Sunucu bir Token üretir ve kullanıcıya verir. Sunucu bu token'ı bir yere kaydetmez (JWT ise).
    
- **Avantaj:** Sunucu durum tutmaz (Stateless).
    
    - 100 tane sunucun da olsa, gelen Token'ın imzasını kontrol etmeleri yeterlidir. Hepsi Ahmet'i tanır.
        
- **Mobil Uyumu:** Token sadece bir metindir (String). Mobil uygulama bunu hafızada saklar ve her isteğin Header'ına ekler. Çok kolaydır.
    

---

### Token Türleri: Opaque vs Self-Contained

İşte burası bir Senior ayrıntısıdır. Her token JWT değildir.

#### A. Self-Contained Token (Kendi Kendine Yeten - Örn: JWT)

Token, veriyi içinde taşır (`{ "id": 1, "role": "admin" }`). Sunucu token'ı okuduğunda veritabanına gitmeye gerek kalmadan kullanıcının kim olduğunu anlar. Hızlıdır ama token boyutu büyüktür.

#### B. Opaque Token (Referans Token - Rastgele String)

Token anlamsız bir karakter dizesidir (Örn: `f8a4b2c1-9e...`).

- **Çalışma Şekli:** Sunucu bu token'ı veritabanına (veya Redis'e) kaydeder.
    
- **Doğrulama:** İstek geldiğinde sunucu veritabanına gider, "Bu token kimin?" diye sorar.
    
- **Avantajı:** Anında iptal edilebilir (Revoke). Kullanıcıyı banladığın an token çöp olur. (JWT'de süresi bitene kadar beklemek zorundasın).
    
- **Dezavantajı:** Her istekte veritabanına gidildiği için JWT'ye göre daha yavaştır.
    

---

### C# ile Opaque (Referans) Token Örneği

JWT'yi zaten incelemiştik. Şimdi veritabanında saklanan **Opaque Token** mantığını kodlayalım. Bu yöntem özellikle "Logout" işleminin çok kritik olduğu (Bankacılık gibi) sistemlerde tercih edilir.

**1. Veritabanı Modeli**

C#

```csharp
public class UserToken
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public string Token { get; set; }     // Örn: Guid String
    public DateTime Expiration { get; set; } // Ne zaman geçersiz olacak?
    public bool IsActive { get; set; }    // Logout olunca false yapacağız
}
```

**2. Token Üretme (Login)**

C#

```csharp
public string GenerateReferenceToken(int userId)
{
    // Rastgele, tahmin edilemez bir string oluştur (Guid)
    var tokenString = Guid.NewGuid().ToString();

    var tokenRecord = new UserToken
    {
        UserId = userId,
        Token = tokenString,
        Expiration = DateTime.Now.AddHours(1),
        IsActive = true
    };

    _context.UserTokens.Add(tokenRecord);
    _context.SaveChanges();

    return tokenString; // Kullanıcıya sadece "abc-123..." dönüyoruz.
}
```

**3. Token Doğrulama (Middleware veya Filter)** Kullanıcı `Authorization: Bearer f8a4...` ile geldiğinde:

C#

```csharp
public bool ValidateToken(string tokenString)
{
    // Veritabanına sor: Böyle bir token var mı?
    var tokenRecord = _context.UserTokens
        .FirstOrDefault(t => t.Token == tokenString);

    // Kontroller
    if (tokenRecord == null) return false; // Token yok
    if (!tokenRecord.IsActive) return false; // Kullanıcı çıkış yapmış
    if (tokenRecord.Expiration < DateTime.Now) return false; // Süresi dolmuş

    return true; // Geçerli
}
```

**4. Çıkış Yapma (Logout - Token Revocation)** JWT'de bunu yapamazsın ama Opaque Token'da çok kolaydır.

C#

```csharp
public void Logout(string tokenString)
{
    var tokenRecord = _context.UserTokens.FirstOrDefault(t => t.Token == tokenString);
    if (tokenRecord != null)
    {
        tokenRecord.IsActive = false; // Token'ı öldür
        _context.SaveChanges();
    }
}
```

---

### Access Token vs. Refresh Token (Kritik İkili)

Token tabanlı sistemlerin güvenlik açığı şudur: **Token çalınırsa ne olur?** Hacker token'ı kaparsa, süresi bitene kadar senin yerine işlem yapabilir.

Bu riski yönetmek için çift token stratejisi kullanılır:

1. **Access Token (Erişim Jetonu):**
    
    - Ömrü çok kısadır (Örn: 15 Dakika).
        
    - API'ye erişmek için kullanılır.
        
    - Çalınsa bile hacker en fazla 15 dakika kullanabilir.
        
2. **Refresh Token (Yenileme Jetonu):**
    
    - Ömrü uzundur (Örn: 7 Gün veya 1 Ay).
        
    - API'ye erişemez. Tek görevi, Access Token'ın süresi bitince sunucudan **yeni bir Access Token** almaktır.
        
    - Genellikle veritabanında saklanır ve "Opaque Token" yapısındadır.
        
    - Eğer kullanıcının hesabı çalınırsa, sunucudan Refresh Token iptal edilir (Revoke). Kullanıcı Access Token süresi (15 dk) bitince sistemden atılır.