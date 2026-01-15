REST, web üzerinden sistemlerin birbirleriyle konuşması için Roy Fielding tarafından 2000 yılında ortaya atılan bir **Mimari Stildir** (Protokol değildir!).

Günümüzde internetin %90'ı bu kurallar bütünüyle çalışır. Eğer bir Junior .NET Developer iş ilanına bakarsan, "RESTful API tecrübesi" maddesini görmeme ihtimalin sıfırdır.

### En Basit Analoji: Restoran

REST API'yi bir **Restoran** gibi düşünebilirsin:

- **Müşteri (Client):** Senin yazdığın Frontend (React) veya Mobil (iOS) uygulaması. Karnı aç (Veriye ihtiyacı var).
    
- **Mutfak (Server/Database):** Yemeğin (Verinin) piştiği yer.
    
- **Menü (API Documentation):** Nelerin sipariş edilebileceğini gösteren liste (Swagger).
    
- **Garson (REST API):** Müşteriden siparişi alır, mutfağa iletir, pişen yemeği müşteriye getirir.
    

Garsonun (REST) kuralları bellidir: "Bana masadan el sallama, standart bir dilde konuş." İşte bu dil **HTTP**'dir.

---

### REST'in 4 Temel Taşı (The 4 Pillars)

Bir API'nin "Ben RESTful'um" diyebilmesi için şu kurallara uyması gerekir:

#### 1. Kaynaklar (Resources - URIs)

REST mimarisinde her şey bir "Kaynak"tır ve isimlerle (Noun) belirtilir. Fiil (Verb) kullanılmaz.

- **Yanlış (Junior Hatası):** `https://api.site.com/getUsers` (Fiil var: get)
    
- **Doğru (RESTful):** `https://api.site.com/users` (Sadece isim)
    

#### 2. HTTP Metotları (Verbs)

Kaynaklar üzerinde ne yapacağını URL ile değil, HTTP metodunla söylersin. (Garsona "Bana su **getir**" veya "Suyu **götür**" demek gibi).

- **GET:** Veriyi okumak için. (Select)
    
- **POST:** Yeni veri oluşturmak için. (Insert)
    
- **PUT:** Veriyi tamamen değiştirmek için. (Update - Replace)
    
- **PATCH:** Verinin bir kısmını değiştirmek için. (Update - Modify)
    
- **DELETE:** Veriyi silmek için. (Delete)
    

#### 3. Stateless (Durumsuzluk)

Burası çok kritiktir. Sunucu, istemcinin (Client) kim olduğunu veya bir önceki istekte ne yaptığını **hatırlamaz**.

- Her istek (Request), sunucunun o işlemi yapması için gereken **tüm bilgileri** (Token, ID, Veri) içermelidir.
    
- Sunucu: "Sen az önce giriş yapmıştın, seni tanıdım" demez. "Token'ını göster, kim olduğuna bakayım" der.
    

#### 4. Standart Cevaplar (HTTP Status Codes)

Sunucu, işlemin sonucunu evrensel kodlarla bildirir. "İşlem başarılı" yazısı göndermek yerine `200` kodu döner.

---

### C# / ASP.NET Core ile REST Uygulaması

Bir .NET projesinde (Web API), REST mimarisi `Controller` sınıfları ile kurulur. Şimdi gerçek bir senaryo üzerinden (Ürün Yönetimi) kodları inceleyelim.

#### 1. GET (Veri Çekme)

Veritabanından veri okur. Asla veritabanında değişiklik yapmaz (**Safe & Idempotent**).

C#

```csharp
[ApiController]
[Route("api/[controller]")] // URL: api/products
public class ProductsController : ControllerBase
{
    // GET: api/products
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = _context.Products.ToList();
        return Ok(products); // 200 OK döner
    }

    // GET: api/products/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _context.Products.Find(id);
        if (product == null)
        {
            return NotFound(); // 404 Not Found döner
        }
        return Ok(product); // 200 OK döner
    }
}
```

#### 2. POST (Veri Ekleme)

Yeni bir kaynak oluşturur. Veri, URL'de değil **Body** (Gövde) kısmında JSON olarak gider.

C#

```csharp
    // POST: api/products
    [HttpPost]
    public IActionResult Create([FromBody] ProductDto productDto)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState); // 400 Bad Request
        }

        var product = new Product { Name = productDto.Name, Price = productDto.Price };
        _context.Products.Add(product);
        _context.SaveChanges();

        // Best Practice: 201 Created döner ve yeni ürünün adresini Header'a koyar.
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
```

#### 3. PUT vs PATCH (Kritik Mülakat Sorusu)

Bu ikisi arasındaki farkı bilmek seni Junior'lıktan kurtarır.

- **PUT (Tam Güncelleme):** Arabanı servise götürdün, "Bunu al, bana şu yeni arabayı ver" dedin. Eski obje tamamen silinir, yenisi konur. Eğer gönderdiğin objede sadece "Ad" var, "Fiyat" yoksa; veritabanındaki fiyat silinip `null` veya `0` olur.
    
- **PATCH (Kısmi Güncelleme):** Arabanı servise götürdün, "Sadece tekerleğini değiştir" dedin. Diğer parçalara (Motor, Koltuk) dokunulmaz.
    

C#

```csharp
    // PUT: api/products/5 (Tüm objeyi bekler)
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Product product)
    {
        if (id != product.Id) return BadRequest();
        
        _context.Entry(product).State = EntityState.Modified; // Hepsini ezdi!
        _context.SaveChanges();
        
        return NoContent(); // 204 No Content (Başarılı ama dönecek veri yok)
    }
```

#### 4. DELETE (Silme)

C#

```csharp
    // DELETE: api/products/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var product = _context.Products.Find(id);
        if (product == null) return NotFound();

        _context.Products.Remove(product);
        _context.SaveChanges();

        return NoContent(); // 204 No Content
    }
```

---

### HTTP Durum Kodları (Status Codes) - REST'in Dili

Frontend geliştiricisiyle anlaşabilmen için bu kodları doğru kullanman şarttır.

- **2xx (Başarılı):**
    
    - `200 OK`: Her şey yolunda (GET için ideal).
        
    - `201 Created`: Yeni kayıt oluştu (POST için ideal).
        
    - `204 No Content`: İşlem tamam ama sana gösterecek bir şeyim yok (DELETE/PUT için ideal).
        
- **4xx (İstemci Hatası - Suç Sende):**
    
    - `400 Bad Request`: Eksik veya hatalı veri gönderdin.
        
    - `401 Unauthorized`: Kimliğin belli değil (Login olmamışsın).
        
    - `403 Forbidden`: Kimliğin belli ama yetkin yok (Admin değilsin).
        
    - `404 Not Found`: İstediğin ID'li ürün yok veya URL yanlış.
        
- **5xx (Sunucu Hatası - Suç Bizde):**
    
    - `500 Internal Server Error`: Kodum patladı (Exception fırlattı), veritabanı çöktü vs.
        

---

### Backend Developer İçin İpuçları

1. **DTO (Data Transfer Object) Kullanımı:** API metodlarında asla veritabanı entity'sini (`Product`) doğrudan dışarı açma veya içeri alma.
    
    - `User` tablosunu direkt dönersen `PasswordHash` alanını da yanlışlıkla JSON içinde gönderirsin.
        
    - Bunun yerine `UserDto` kullan, sadece `Name` ve `Email` alanlarını barındırsın.
        
2. **Versioning (Versiyonlama):** API canlıya çıktıktan sonra yapıyı değiştiremezsin (Mobil uygulamayı güncellemeyen kullanıcılar patlar).
    
    - Bu yüzden URL'lerinde versiyon kullan: `api/v1/products`, `api/v2/products`.
        
3. **JSON Standartı:** REST API'ler günümüzde veri formatı olarak XML yerine neredeyse tamamen **JSON** kullanır. Hafiftir ve JavaScript ile doğuştan uyumludur. C# tarafında `System.Text.Json` veya `Newtonsoft.Json` bu dönüşümü otomatik yapar.