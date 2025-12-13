
Çoğu geliştirici "Ben REST API yazdım" der ama aslında yazdığı şey sadece "HTTP üzerinden çalışan bir fonksiyon"dur. Gerçek bir REST mimarisi, Roy Fielding'in 2000 yılında doktora tezinde yazdığı katı kurallara (Constraints) dayanır.1

Bu konuyu "URL'e istek at, JSON gelsin" basitliğinden çıkarıp, **Richardson Olgunluk Modeli (Maturity Model)** ve **Idempotency (Etkisizlik)** gibi kıdemli mühendislik kavramlarıyla inceleyeceğiz.

---

### 1. REST Nedir? (Felsefe ve İsim Analizi)

**REST (Representational State Transfer)**, bir protokol değil (SOAP gibi), bir **Mimari Stildir.2**

İsmini analiz edelim, çünkü sırrı burada saklı:

1. **Representational (Temsili):** Sunucudaki "Ürün" veritabanında bir satırdır. Ama istemciye (Client) gönderirken biz onun **temsili** bir halini (JSON veya XML) göndeririz.
    
2. **State Transfer (Durum Transferi):** Burası en kritik yer. Sunucu, istemcinin o an hangi sayfada olduğunu, sepetinde ne olduğunu **tutmaz**. İstemci, her istekte kendi durumunu (State) sunucuya transfer eder.
    

**Mühendislik Notu:** REST, dağıtık sistemlerin (Web) ölçeklenebilir olması için tasarlanmıştır.

---

### 2. REST'in 6 Kutsal Kuralı (Constraints)

Bir servise "RESTful" diyebilmek için şu kurallara uyması gerekir:

1. **Client-Server:** İstemci (Mobil/Web) ve Sunucu (API) birbirinden tamamen bağımsızdır. Birini değiştirmek diğerini bozmamalıdır.
    
2. **Stateless (Durumsuzluk):** **En önemlisi.** Sunucu, "Ahmet az önce login oldu, oturumu açık" diye bir bilgiyi RAM'de tutmaz. Her istek (Request), kimlik bilgisi (Token) dahil her şeyi baştan göndermelidir.
    
    - _Neden?_ Sunucu A çökerse, Sunucu B isteği karşılayabilir çünkü hafızada kullanıcıya özel veri yoktur. Cloud sistemlerin temeli budur.
        
3. **Cacheable:** Sunucu, cevabın önbelleğe alınıp alınamayacağını belirtmelidir (`Cache-Control` header).
    
4. **Uniform Interface (Tek Tip Arayüz):** API'nin URL yapısı ve metodları standart olmalıdır. (Detayına aşağıda gireceğiz).
    
5. **Layered System:** İstemci, doğrudan sunucuya mı yoksa aradaki bir Load Balancer'a (Yük Dengeleyici) mı bağlandığını bilmez/bilmemelidir.
    
6. **Code on Demand (Opsiyonel):** Sunucu, istemciye çalıştırılabilir kod (örn: Java Applet, JS) gönderebilir. (Günümüzde pek kullanılmaz).
    

---

### 3. Richardson Maturity Model (Senin API'n Ne Kadar REST?)

Her HTTP servisi REST değildir. Leonard Richardson, API'leri 4 seviyeye ayırmıştır:

- **Level 0 (The Swamp of POX):** HTTP'yi sadece tünel olarak kullanır. Tek bir URL vardır (`/api/service`) ve genelde sadece `POST` kullanılır. (Eski SOAP mantığı).
    
- **Level 1 (Resources):** URL'ler kaynaklara bölünmüştür (`/api/users`, `/api/products`). Ama hala her şey için `POST` veya `GET` kullanılır.
    
- **Level 2 (HTTP Verbs - Bizim Hedefimiz):** HTTP fiilleri (GET, POST, PUT, DELETE) doğru anlamlarıyla kullanılır.3 Durum kodları (200, 404, 201) doğru döner. Sektör standardı budur.
    
- **Level 3 (HATEOAS - Hypermedia):** API cevabının içinde, o veriyle yapılabilecek diğer işlemlerin linkleri de döner. (Çok gelişmiş ama uygulaması zor olduğu için nadir kullanılır).
    

---

### 4. HTTP Metotları ve Anlamsal (Semantic) Farklar

Bir .NET Developer olarak Controller içinde bu metotları doğru eşleştirmelisin.

- **GET (Read):** Veri okur. **Safe (Güvenli)** metodur, veritabanını asla değiştirmez.
    
- **POST (Create):** Yeni kaynak yaratır.
    
- **PUT (Full Update):** Bir kaynağı **tamamen** değiştirir.4 ID:5 olan ürünü gönderdiğin yeni haliyle ezer.
    
- **PATCH (Partial Update):** Bir kaynağı **kısmen** değiştirir.5 Sadece "Fiyat" alanını gönderirsen, sadece fiyat değişir.
    
- **DELETE:** Kaynağı siler.6
    

#### Kritik Kavram: Idempotency (Etkisizlik)

Bir işlemi 1 kere yapmakla 100 kere yapmak arasında fark yoksa o işlem **Idempotent**'tir.

- **GET:** Idempotent'tir. 100 kere de çağırsan sonuç aynıdır, veri değişmez.
    
- **PUT / DELETE:** Idempotent'tir. "ID:5'i sil" komutunu 100 kere gönderirsen, ilkinde silinir, diğerlerinde "Zaten yok" der veya aynısı olur. Sonuç (o verinin olmaması) değişmez.
    
- **POST:** **Idempotent DEĞİLDİR.7** Eğer "Sipariş Oluştur" butonuna yanlışlıkla 2 kere basarsan, 2 farklı sipariş oluşur. (Bunu önlemek için özel "Idempotency Key" mekanizmaları kurulur).8
    

---

### 5. Kaynak İsimlendirme (Resource Naming Best Practices)

REST API tasarlarken en sık yapılan hatalar URL yapısında olur.

- **Kural 1: İsim (Noun) Kullan, Fiil (Verb) Değil.**
    
    - _Yanlış:_ `/api/getAllUsers`, `/api/createUser`, `/api/deleteUser?id=5`
        
    - _Doğru:_ `/api/users` (Metot GET, POST veya DELETE olur, URL değişmez).
        
- **Kural 2: Çoğul İsim Kullan.**
    
    - _Tercihen:_ `/api/users/5` (Kullanıcılar koleksiyonunun 5 numaralı elemanı).
        
- **Kural 3: Hiyerarşi.**
    
    - `1` numaralı kullanıcının siparişlerini getirmek için:
        
    - `/api/users/1/orders`
        

---

### 6. ASP.NET Core Entegrasyonu

C# tarafında bu işler `Controller` içinde şöyle yönetilir:

C#

```csharp
[Route("api/[controller]")] // URL: api/products
[ApiController]
public class ProductsController : ControllerBase
{
    // GET: api/products
    [HttpGet] 
    public IActionResult GetAll() { ... }

    // GET: api/products/5
    [HttpGet("{id}")] 
    public IActionResult GetById(int id) { ... }

    // POST: api/products
    [HttpPost] 
    public IActionResult Create(Product p) 
    { 
        // 201 Created döner ve "Location" header'ında yeni ürünün linkini verir
        return CreatedAtAction(nameof(GetById), new { id = p.Id }, p); 
    }
}
```

---

### 1. REST API & RESTful Architecture

**🧒 6 Yaşındaki Çocuğa (Otomat Makinesi Analojisi):** "REST API'yi, AVM'lerdeki yiyecek otomatlarına benzetebiliriz. Bu makinenin kuralları çok nettir ve dünyanın her yerinde aynıdır (Uniform Interface):

- İçindekileri görmek için camdan bakarsın (**GET**). Bunu 100 kere de yapsan makinedeki ürünler eksilmez (**Safe & Idempotent**).
    
- Para atıp bir çikolata istersen, makine sana yeni bir ürün verir (**POST**).
    
- Eğer makineden uzaklaşıp 5 dakika sonra geri gelirsen, makine seni tanımaz. 'Az önce gelen çocuksun, paranı biliyorum' demez. Paranı tekrar atman gerekir. İşte buna **Stateless (Durumsuzluk)** denir. Makine hafızasında seni tutmaz, her işlem yeni bir başlangıçtır."
    

**👨‍💼 Mülakatta Yöneticiye (Abstraction):** "REST, dağıtık sistemlerin birbiriyle iletişim kurması için Roy Fielding tarafından belirlenmiş, internetin ölçeklenebilirliğini temel alan bir **Mimari Stildir (Architectural Style)**. Bizim için iki kritik avantajı vardır:

1. **Stateless (Durumsuz) Yapı:** Sunucu, istemcinin (Client) o anki durumunu (oturum bilgisini) RAM'de tutmaz. Her istek, kimlik bilgisi dahil ihtiyacı olan her şeyi paketin içinde taşır. Bu sayede sunucularımızı çok kolay bir şekilde yatayda ölçekleyebiliriz (Horizontal Scaling). Bir sunucu çökerse, diğer sunucu isteği karşılayabilir çünkü hafızasında kullanıcıya özel bir veri yoktur.
    
2. **Standartlaşma (Uniform Interface):** HTTP metotlarını (GET, POST, PUT, DELETE) evrensel standartlarda kullandığımız için, Frontend ve Mobil ekipleriyle entegrasyon maliyetimiz minimuma iner. Dokümantasyon ihtiyacı azalır."

