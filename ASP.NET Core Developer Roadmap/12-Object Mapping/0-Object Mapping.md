Real-Time dünyasındaki soket ve state yönetimini geride bıraktık. Şimdi uygulamanın **veri omurgasını** oluşturan, katmanlı mimarinin (N-Layer Architecture) en büyük hamalı olan **Object Mapping (Nesne Eşleme)** konusuna geçiyoruz.

Pek çok geliştirici Object Mapping'i sadece "A sınıfındaki veriyi B sınıfına kopyalamak" zanneder. Oysa bu konu; **Veri Güvenliği**, **API Performansı** ve **Abstraction (Soyutlama)** stratejisinin tam kalbidir.

Attığın metinde AutoMapper, Mapster gibi kütüphanelerden bahsedilmiş. Biz bu konuyu sadece "nasıl kullanılır" değil, **Manual Mapping vs Auto Mapping** savaşı, **Projection (Yansıtma)** performansı ve **Flattening (Düzleştirme)** teknikleri üzerinden, tek seferde ve eksiksiz bir mühendislik derinliğinde inceleyelim.

---

### 1. Neden İhtiyacımız Var? (Entity vs DTO)

Veritabanı nesnelerini (Entity) doğrudan API'den dışarı açmak (**Exposing Domain Models**) bir yazılım suçudur.

- **Güvenlik:** `User` tablosunda `PasswordHash` veya `Salt` alanları vardır. Bunları yanlışlıkla dışarı açarsan hacklenirsin.
    
- **Döngüsel Referans (Circular Reference):** `User` -> `Orders` -> `User`... JSON serileştirici sonsuz döngüye girer.
    
- **Over-fetching (Gereksiz Veri):** Mobil uygulama sadece `Ad` ve `Soyad` istiyorsa, 50 sütunlu `User` tablosunu göndermek ağ israfıdır.
    

Bu yüzden **DTO (Data Transfer Object)** kullanırız. Veriyi `Entity`'den `DTO`'ya aktarma işine **Object Mapping** denir.

---

### 2. Yöntemler: Manual vs Automated

Bir mimar olarak proje başında şu kararı vermelisin:

#### A. Manual Mapping (El İle)

C#

```cs
var userDto = new UserDto {
    Name = user.Name,
    Email = user.Email
    // 50 tane daha alan varsa geçmiş olsun...
};
```

- **Avantajı:** En hızlısıdır (Compile-time). Çalışma zamanında (Runtime) sürpriz yapmaz. Debug etmesi çok kolaydır.
    
- **Dezavantajı:** Kod kirliliği (Boilerplate). Yeni bir alan eklediğinde maplemeyi unutursan veri boş gider.
    

#### B. Automated Mapping (AutoMapper / Mapster)

C#

```cs
var userDto = _mapper.Map<UserDto>(user);
```

- **Avantajı:** Kod temizdir. Convention (Kural) bazlı çalışır (Adı aynıysa eşleştirir).
    
- **Dezavantajı:** Reflection kullandığı için (özellikle AutoMapper) çok az bir performans maliyetine sahiptir. Debug etmesi zordur ("Bu veri neden gelmedi?" diye kütüphane koduna bakamazsın).
    

---

### 3. Sektör Standardı: AutoMapper

.NET dünyasının en eskisi ve en popüleri **AutoMapper**'dır.

Profile Yapısı (Düzen):

Mapping kuralları Program.cs içine değil, özel Profile sınıflarına yazılır.

C#

```cs
public class UserProfile : Profile
{
    public UserProfile()
    {
        CreateMap<User, UserDto>()
            .ForMember(dest => dest.TamAd, opt => opt.MapFrom(src => src.Ad + " " + src.Soyad)) // Custom Logic
            .ReverseMap(); // İki yönlü (DTO -> Entity) çalışsın
    }
}
```

Flattening (Düzleştirme) Büyüsü:

AutoMapper'ın en sevilen özelliğidir.

- **Entity:** `Order.Customer.Name` (İç içe nesneler).
    
- **DTO:** `OrderDto` içinde `CustomerName` diye bir property varsa, AutoMapper bunu otomatik algılar ve eşleştirir. Hiçbir ayar yapmana gerek kalmaz.
    

---

### 4. Performans Kralı: Mapster

Eğer performans senin için her şeyse (High Frequency Trading gibi), AutoMapper yerine **Mapster** seçersin.

- **Farkı:** AutoMapper çalışma zamanında (Runtime) Reflection kullanır. Mapster ise kod derlenirken (Compile Time) veya ilk çalışmada **Expression Tree** üreterek çalışır. Yani sanki sen elle yazmışsın gibi hızlıdır.
    
- **Kullanım:** Neredeyse AutoMapper ile aynıdır ama daha hafiftir.
    
- **Syntax:** `var dto = user.Adapt<UserDto>();`
    

---

### 5. En Kritik Mühendislik Konusu: `ProjectTo` (Projection)

Burası bir Senior Developer ile Junior'ı ayıran çizgidir.

**Senaryo:** `Users` tablosunda 1 milyon kayıt ve 50 sütun var. Listeleme ekranında sadece `Ad` ve `Soyad` göstereceksin.

**Yanlış Yöntem (In-Memory Mapping):**

C#

```cs
// 1. Veritabanından TÜM satırları ve TÜM sütunları çeker (SELECT * FROM Users).
var users = _context.Users.ToList(); 
// 2. RAM'de mapleme yapar.
var dtos = _mapper.Map<List<UserDto>>(users); 
```

- **Sonuç:** Veritabanı ve Ağ trafiği patlar. RAM şişer.
    

**Doğru Yöntem (Projection - `IQueryable`):**

C#

```cs
// AutoMapper'ın Queryable Extensions özelliği
var dtos = _context.Users
    .ProjectTo<UserDto>(_mapper.ConfigurationProvider) // Sihir burada!
    .ToList();
```

- **Sonuç:** AutoMapper, DTO'ya bakar. Sadece `Ad` ve `Soyad` alanlarını görür.
    
- **Oluşan SQL:** `SELECT Ad, Soyad FROM Users`
    
- Veri RAM'e gelmeden, SQL Server seviyesinde filtrelenir. Muazzam performans artışı sağlar.
    

---

### 6. Mapping Anti-Patterns (Yapılmaması Gerekenler)

1. **Business Logic in Mapping:** Mapping sırasında karmaşık hesaplamalar (Vergi hesabı vb.) yapma. Mapper'ın görevi sadece veri taşımaktır. İş mantığı Domain Service'te olmalıdır.
    
2. **Entity in Controller:** "Sadece 2 alan var, DTO yazmaya üşendim" deyip Controller'dan Entity dönme. Bir gün o Entity'e hassas veri eklendiğinde unutursun ve sızar.
    
3. **Ignoring Validations:** `AssertConfigurationIsValid()` metodunu Test (Unit Test) aşamasında mutlaka çağır. Bu metot, "DTO'da bir alan var ama Entity'de karşılığı yok, unuttun mu?" diye kontrol eder. Canlıya çıkmadan hatayı yakalarsın.
    

---

**🧒 6 Yaşındaki Çocuğa (Cüzdan Analojisi):** "Babanın cebindeki cüzdanı düşün (**Entity**). Bu cüzdanın içinde her şey var: Paralar, kredi kartları, kimlik, belki gizli bir fotoğraf. Eğer bakkal amca babana 'Sen kimsin?' diye sorarsa, baban cüzdanı komple bakkala vermez, değil mi? İçindeki paraları alabilirler! Baban cüzdanı açar, içinden sadece kimliği çıkarır ve bakkala gösterir (**DTO**). İşte **Object Mapping**, o cüzdanın içinden sadece gerekli olanı seçip alma işidir. Bunu elle yaparsan yavaş olur. **AutoMapper** ise sihirli bir makine gibidir; cüzdanı makineye atarsın, makine sana saniyesinde sadece kimliği fırlatır."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "Katmanlı mimaride, veritabanı nesnelerinin (Entities) doğrudan sunum katmanına (API) açılması; güvenlik zaafiyetlerine (Over-exposure) ve gereksiz veri transferine (Over-fetching) yol açar. Bu nedenle veri transferinde mutlaka **DTO (Data Transfer Object)** deseni uygulanır. Nesne dönüşümleri için mimari yaklaşım şöyledir:

- **Boilerplate Yönetimi:** Kod tekrarını ve kirliliğini önlemek adına, manuel eşleme yerine **AutoMapper** veya **Mapster** gibi kütüphanelerle 'Convention-based' (Kural bazlı) otomatik eşleme tercih edilir.
    
- **Performance Optimization (Projection):** Mapping işleminin en kritik noktası bellektir. Veriyi veritabanından çektikten sonra RAM'de dönüştürmek yerine; **`ProjectTo`** gibi yöntemler kullanılarak, eşlemenin `IQueryable` seviyesinde yapılması sağlanır. Böylece veritabanına giden SQL sorgusu sadece istenen sütunları (`SELECT Name` gibi) çekecek şekilde optimize edilir.
    
- **Compile-Time Mapping:** Yüksek performans gerektiren (High-Frequency) senaryolarda ise, Reflection maliyetinden kaçınmak için çalışma zamanı yerine derleme zamanında kod üreten **Mapster** gibi çözümler mimariye dahil edilir."