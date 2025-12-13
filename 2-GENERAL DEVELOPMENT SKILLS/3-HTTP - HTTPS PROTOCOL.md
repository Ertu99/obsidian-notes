

Bir .NET Web API geliştireceğin zaman, aslında yapacağın işin %90'ı **HTTP protokolünü yönetmek** olacak. Eğer bu konuyu "sular seller gibi" bilirsen, yazdığın API'lar sağlam, güvenli ve standartlara uygun olur. Bilmezsen, sadece "çalışan ama hatalı" kodlar yazarsın.



---

### 1. HTTP Nedir? (İnternetin Dili)

**HTTP (HyperText Transfer Protocol)**, İstemci (Client - Tarayıcı veya Mobile App) ile Sunucu (Server - Senin yazacağın .NET uygulaması) arasındaki konuşma kurallarıdır.1

Şöyle düşün:

- **Client:** Restorandaki müşteri.
    
- **Server:** Mutfak.
    
- **HTTP:** Garson.
    

Müşteri mutfağa direkt giremez. Garsona (HTTP) bir sipariş verir (**Request**), garson bunu belirli bir formatta mutfağa iletir, mutfak yemeği hazırlar ve garson yemeği (veya "malzeme kalmadı" bilgisini) müşteriye geri getirir (**Response**).2

Bu iletişim **Stateless (Durumsuz)** bir iletişimdir. Yani sunucu, bir önceki isteği hatırlamaz. Her istek, sıfırdan yapılmış yeni bir başlangıçtır.

![HTTP request response cycle resmi](https://encrypted-tbn1.gstatic.com/licensed-image?q=tbn:ANd9GcTekePMTuDcoodx3jamXfbIaDzuudptbvAt26X59A5etohy3C7jek7zYCUcWlHp3UeENF-rk6W0gMmXY2rv2zHULP9YAjdaQdtlvMTur2LsLNS-GX0)


---

### 2. Bir HTTP İsteğinin Anatomisi (Request)

Sen bir siteye girdiğinde veya API'ya istek attığında, kabloların içinden aslında şöyle bir metin (text) akar:

HTTP

```
POST /api/users HTTP/1.1
Host: google.com
Content-Type: application/json
Authorization: Bearer sakli_sifre_token

{
    "ad": "Ertuğrul",
    "yas": 26
}
```

Bunu parçalayalım:

1. **Metot (Verb):** `POST`. "Ne yapmak istiyorsun?" sorusunun cevabı.3
    
2. **Path (Yol):** `/api/users`. "Nereye gidiyorsun?"
    
3. **Headers (Başlıklar):** Mektubun zarfı. Kimlik bilgileri (`Authorization`), içeriğin tipi (`Content-Type`) gibi meta veriler buradadır.
    
4. **Body (Gövde):** Mektubun kendisi. Gönderdiğin veriler (JSON) burada taşınır.
    

---

### 3. HTTP Metotları (Backend'in 4 Atlısı)

Bir Backend Developer olarak C# kodunda sürekli bu metodları karşılayacaksın. Bunlara **CRUD** operasyonları denir.

- **GET:** Veri **okumak** için. (Asla veritabanında değişiklik yapmaz).
    
    - _Örn:_ Ürün listesini getir.
        
- **POST:** Yeni veri **yaratmak** için.4
    
    - _Örn:_ Yeni kullanıcı kaydet.
        
- **PUT:** Veriyi **tamamen değiştirmek/güncellemek** için.
    
    - _Örn:_ Ürünün adını, fiyatını, stoğunu komple güncelle.
        
- **DELETE:** Veri **silmek** için.
    
    - _Örn:_ Kullanıcıyı sil.
        

_(Bir de **PATCH** vardır, verinin sadece bir kısmını -mesela sadece şifresini- güncellemek için kullanılır.)_5

---

### 4. HTTP Durum Kodları (Status Codes)

Server cevabı döndüğünde, sadece veriyi değil, işlemin sonucunu belirten 3 haneli bir sayı döner. Bunu yanlış kullanırsan Frontend developer sana küser.

- **2xx (Başarılı):**
    
    - `200 OK`: Her şey yolunda.6
        
    - `201 Created`: Başarılı ve yeni bir kayıt oluşturuldu (POST işlemlerinde döner).
        
- **3xx (Yönlendirme):**
    
    - `301 Moved Permanently`: Sayfa başka yere taşındı.
        
- **4xx (Sen Hatalısın - Client Error):**
    
    - `400 Bad Request`: Eksik veya hatalı veri gönderdin.
        
    - `401 Unauthorized`: Giriş yapmamışsın, kimsin bilmiyorum.
        
    - `403 Forbidden`: Giriş yapmışsın ama yetkin yok (Müdürün odasına girmeye çalışan stajyer).
        
    - `404 Not Found`: Böyle bir sayfa/veri yok.
        
- **5xx (Ben Hatalıyım - Server Error):**
    
    - `500 Internal Server Error`: Kodum patladı, veritabanı çöktü vs. (Senin suçun değil, backend'in suçu).
        

---

### 5. HTTPS: Güvenlik Katmanı

**HTTPS = HTTP + SSL/TLS**

Normal HTTP'de veriler **"Plain Text" (Düz Metin)** olarak gider. Yani aynı Wi-Fi ağındaki bir hacker, `Wireshark` gibi bir programla senin gönderdiğin şifreyi, kredi kartı numarasını kabak gibi okuyabilir.

HTTPS'de ise **SSL/TLS Handshake** (El Sıkışma) devreye girer:

1. Browser ve Server konuşup bir şifreleme anahtarı üzerinde anlaşır.
    
2. Veriler şifrelenir (Encrypt).
    
3. Hacker araya girse bile `x8s!_a2...` gibi anlamsız karakterler görür.
    

Backend developer olarak senin görevin, sunucuda (IIS, Nginx veya Kestrel) SSL sertifikasını aktif etmektir.

---

### 6. C# .NET Core Bağlantısı

Şu an çok havada kalmasın, bu anlattıklarımın C#'ta neye denk geldiğini göstereyim:

C#

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // HTTP GET isteği gelirse bu metot çalışır
    // Cevap olarak 200 OK döner
    [HttpGet]
    public IActionResult GetUsers()
    {
        var users = _service.GetAll();
        return Ok(users); // 200 OK
    }

    // HTTP POST isteği gelirse bu metot çalışır
    [HttpPost]
    public IActionResult CreateUser(User user)
    {
        _service.Add(user);
        return Created("", user); // 201 Created
    }
}
```

Gördüğün gibi, C# tarafında her şey bu protokole göre dizayn edilmiştir.

---

**🧒 6 Yaşındaki Çocuğa:** "Sınıfta arkadaşına bir not göndermek istediğini düşün. Eğer notu kağıda yazıp, katlamadan elden ele gönderirsen, yoldaki diğer çocuklar (Ali, Ayşe) kağıdı okuyabilir. İşte bu **HTTP**'dir. Hızlıdır ama sırrını herkes görebilir. Ama eğer notu yazıp kilitli bir kutuya koyarsan ve anahtar sadece arkadaşında varsa, yoldaki kimse kutunun içinde ne olduğunu göremez. Sadece kutuyu taşır. İşte bu da **HTTPS**'tir. İnternette şifrelerimizi ve özel bilgilerimizi gönderirken, kötü adamlar okuyamasın diye hep o kilitli kutuyu (HTTPS) kullanırız."

**👨‍💼 Mülakatta Yöneticiye (Abstraction):** "HTTP, uygulamamızın dünyayla iletişim kurmasını sağlayan **evrensel dildir (Application Layer Protocol)**. İstemci (Client) ve Sunucu (Server) arasındaki veri alışverişini standart bir formatta (Request/Response) yönetir. Bir Mimar olarak HTTP'de dikkat ettiğim iki temel husus vardır:

1. **Stateless (Durumsuz) Olması:** Sunucunun geçmişi hatırlamaması, sunucuyu ölçeklememizi (Scaling) kolaylaştırır. Her istek kendi ayakları üzerinde durur.
    
2. **Güvenlik (HTTPS):** Veri güvenliği ve bütünlüğü (Data Integrity) için, verilerin transfer sırasında şifrelenmesini sağlayan SSL/TLS katmanını (HTTPS) mutlaka kullanırız. Bu artık bir tercih değil, endüstri standardıdır."