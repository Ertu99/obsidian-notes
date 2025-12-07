
# Entity Framework Core: Derinlemesine Mimari Analiz

Tanımda dendiği gibi EF Core, "Hafif, platformlar arası (Cross-Platform) ve açık kaynaklı" bir ORM aracıdır. Ama onu sadece "SQL yazmaktan kurtaran araç" olarak görmek, Ferrari'yi tüp taktırıp kullanmak gibidir.

EF Core, .NET uygulamaları için bir **Data Access (Veri Erişim)** katmanıdır ve **Repository** ile **Unit of Work** tasarım desenlerinin (Design Patterns) canlı kanlı halidir.

### 1. EF Core'un Kalbi: `DbContext`

EF Core ile yaptığın her şey `DbContext` sınıfından türeyen bir sınıf (örneğin `AppDbContext`) üzerinden döner. Peki bu sınıf aslında nedir?

- **Unit of Work (İş Birimi) Desenidir:** `DbContext`, bir işlem (transaction) boyunca yapılan tüm değişiklikleri (ekleme, silme, güncelleme) hafızasında tutar. Sen `SaveChanges()` diyene kadar veritabanına tek bir byte bile gitmez. `SaveChanges()` dediğin an, biriktirdiği tüm işleri **tek bir transaction** içinde veritabanına gömer. Bu, veri bütünlüğü (Data Integrity) sağlar.
    
- **Database Session (Veritabanı Oturumu):** Veritabanı ile açılan aktif bir bağlantıyı temsil eder. Bu yüzden `DbContext` nesneleri **hafif (lightweight)** olmalı ve iş bitince hemen yok edilmelidir (Disposable). Asla static veya singleton yapılmaz!
    

### 2. Tablo Temsilcileri: `DbSet<T>`

`DbContext` içindeki `public DbSet<User> Users { get; set; }` satırı, veritabanındaki `Users` tablosunun C# tarafındaki vekilidir (Proxy).

- **Sadece Koleksiyon Değildir:** `DbSet`, bir `List<>` gibi davranmaz. O bir **`IQueryable`** kaynağıdır. Yani sen onun üzerine LINQ yazdığında, bu kodlar C# belleğinde çalışmaz; SQL'e çevrilmek üzere hazırlanır.
    

---

### 3. EF Core Nasıl Çalışır? (Sorgu Çeviri Hattı)

Sen `context.Users.Where(u => u.Age > 18).ToList()` yazdığında arka planda ne olur? Bu süreç 4 aşamadan oluşur:

1. **LINQ Expression Tree:** C# derleyicisi yazdığın lambda ifadesini (`u => u.Age > 18`) bir "İfade Ağacı"na dönüştürür. Bu, kodun mantıksal haritasıdır.
    
2. **Translation (Çeviri):** EF Core Provider'ı (SQL Server, PostgreSQL vb.), bu ağacı alır ve kendi diline (SQL) çevirmeye çalışır.
    
    - _Kritik Detay:_ Her C# kodu SQL'e çevrilemez. Örneğin `Where(u => u.Name.Split(',')[0] == "Ahmet")` dersen, SQL Server `Split` metodunu bilmediği için EF Core hata verir veya veriyi belleğe çekip orada işlemeye çalışır (Client-side evaluation). Bu performans için büyük risktir.
        
3. **Optimization:** Oluşan SQL sorgusu optimize edilir.
    
4. **Execution (Çalıştırma):** Sorgu veritabanına gönderilir, sonuçlar (ResultSet) gelir ve bu sonuçlar tekrar C# nesnelerine (User) dönüştürülür (Materialization).
    

---

### 4. Geliştirme Yaklaşımları (Workflows)

EF Core kullanırken projeye başlama stratejin hayati önem taşır.

#### A. Code First (Kod Öncelikli) - Modern Standart

Önce C# classlarını (Entity) yazarsın, veritabanını EF Core oluşturur.

- **Avantajı:** Veritabanı şeması üzerinde tam kontrol sağlar. Versiyonlama (Migrations) ile DB değişikliklerini Git üzerinde takip edebilirsin.
    
- **Nasıl Çalışır?** `Snapshot` mekanizması vardır. EF Core, veritabanının son halini bilir. Sen yeni bir property eklediğinde aradaki farkı hesaplar ve `ALTER TABLE` scripti üretir.
    

#### B. Database First (Veritabanı Öncelikli)

Önce SQL tarafında tabloları elle açarsın, EF Core bu tablolara bakarak C# classlarını oluşturur (`Scaffold-DbContext`).

- **Dezavantajı:** Üretilen classlarda manuel değişiklik yapamazsın (çünkü tekrar scaffold yapınca silinir). Partial class kullanman gerekir.
    

---

### 5. `Change Tracker` (Değişiklik İzleyicisi) - Gizli Ajan

EF Core'un en karmaşık ve en güçlü özelliğidir. Veritabanından bir veri çektiğinde, EF Core o nesnenin bir kopyasını **Change Tracker** denen hafıza alanında saklar.

Her nesnenin bir **EntityState** (Durum) bayrağı vardır:

1. **Unchanged:** Veritabanından geldi, henüz dokunmadın.
    
2. **Added:** Yeni eklendi (`.Add()`), henüz DB'de yok.
    
3. **Modified:** Bir alanı değiştirdin.
    
4. **Deleted:** Silinmek üzere işaretlendi (`.Remove()`).
    

Sen `SaveChanges()` dediğinde, EF Core bu listeyi gezer:

- `Added` olanlar için -> `INSERT INTO...`
    
- `Modified` olanlar için -> `UPDATE...`
    
- Deleted olanlar için -> DELETE FROM...
    
    sorgularını oluşturur ve gönderir.
    

**Performans İpucu:** `AsNoTracking()` kullandığında işte bu mekanizmayı devre dışı bırakırsın. "İzleme yapma, sadece veriyi ver" dersin.

---

### 6. Bilinmeyen Özellikler (Advanced Features)

Çoğu tutorial'ın pas geçtiği ama profesyonel projelerde ihtiyaç duyacağın özellikler:

#### A. Shadow Properties (Gölge Özellikler)

C# sınıfında (`User.cs`) olmayan ama veritabanı tablosunda olan sütunlar tanımlayabilirsin.

- **Örnek:** `CreatedDate` veya `LastUpdated` alanlarını entity sınıfına koymak istemiyorsun ama DB'de olsun istiyorsun. EF Core bunu `context.Entry(user).Property("LastUpdated").CurrentValue = DateTime.Now;` şeklinde yönetebilir.
    

#### B. Global Query Filters (Küresel Filtreler)

Uygulamanda "Soft Delete" (Veriyi silmeyip `IsDeleted = true` yapma) kullanıyorsan, her sorguda `.Where(x => !x.IsDeleted)` yazmak eziyettir.

- **Çözüm:** `OnModelCreating` metodunda global filtre tanımlarsın. Artık `context.Users.ToList()` dediğinde EF Core otomatik olarak `WHERE IsDeleted = 0` ekler.
    

#### C. Concurrency Tokens (Eşzamanlılık Jetonu)

İki kullanıcı aynı anda aynı kaydı güncellemeye çalışırsa ne olur? (Son yazan kazanır - Last Write Wins).

- **Çözüm:** Bir sütunu (örneğin `RowVersion`) `[ConcurrencyCheck]` olarak işaretlersin.
    
- EF Core güncelleme yaparken `UPDATE ... WHERE Id=1 AND RowVersion=EskiVersion` sorgusu atar. Eğer RowVersion değişmişse (başkası güncellemişse) `DbUpdateConcurrencyException` hatası fırlatır.
    

---

### 7. Yapılandırma Yöntemleri: Data Annotations vs Fluent API

EF Core'a "Bu alan zorunlu", "Bu alanın tipi VARCHAR(50)" gibi ayarları nasıl söylersin?

1. **Data Annotations (Basit):** Entity sınıfının tepesine etiket yapıştırırsın.
    
    C#
    
    ```csharp
    [Required]
    [MaxLength(50)]
    public string Name { get; set; }
    ```
    
    - _Sınırı:_ Her ayarı yapamazsın (örn: Composite Key). Entity sınıfını kirletir.
        
2. **Fluent API (Profesyonel):** `DbContext` içindeki `OnModelCreating` metodunda C# koduyla ayar yaparsın.
    
    C#
    
    ```csharp
    modelBuilder.Entity<User>()
        .Property(u => u.Name)
        .IsRequired()
        .HasMaxLength(50);
    ```
    
    - _Gücü:_ Veritabanı kurallarını Entity sınıfından ayırır (Separation of Concerns). Çok daha yeteneklidir.
        

---

### Entities (Varlıklar) - POCO ve Domain Model

Metinde "objects in an application" olarak geçen kısımdır.

**Mühendislik Derinliği:** EF Core'da Entity'ler genellikle **POCO (Plain Old CLR Object)** olarak tasarlanır. Yani, sınıfın içinde EF Core'a dair hiçbir kütüphane (`using Microsoft.EntityFrameworkCore;`) olmamalıdır. Bu, **Clean Architecture** prensibidir. Domain katmanı veritabanından habersiz olmalıdır.

- **Anemic Domain Model (Kansız Model):** Sadece `get; set;` barındıran, içi boş sınıflar. (Genelde Anti-Pattern kabul edilir ama yaygındır).
    
- **Rich Domain Model (Zengin Model):** Kendi içinde validation (doğrulama) ve business logic barındıran sınıflar. EF Core, "Backing Fields" özelliği ile bu zengin modelleri (Private set'leri olsa bile) yönetebilir.
    

### Mapping (Haritalama) - Köprü Mühendisliği

"Maps the objects... to the database tables" kısmı.

C# tipleri ile SQL tipleri birebir uyuşmaz.

- C#: `string` (Sınırsız olabilir) -> SQL: `NVARCHAR(MAX)` mı `VARCHAR(50)` mi?
    
- C#: `decimal` -> SQL: `MONEY` mi `DECIMAL(18,2)` mi?
    

EF Core burada 3 yöntem sunar:

1. **Conventions (Varsayılanlar):** `Id` isimli bir property görürse "Ha, bu Primary Key'dir" der.
    
2. **Data Annotations (Etiketler):** `[MaxLength(50)]` gibi attribute'lar.
    
3. **Fluent API (En Güçlüsü):** `OnModelCreating` içinde ince ayar.
    

**Gelişmiş Özellik: Shadow Properties** Bazen veritabanında bir sütun olsun istersin (örn: `LastUpdated`), ama C# Entity sınıfında bu property gözüksün istemezsin (kod kirliliği olmasın). Mapping katmanında bunu "Shadow Property" olarak tanımlarsın. EF Core bunu arka planda yönetir, sen kodda görmezsin.

### Context (Bağlam) - Oturum Yönetimi

Metinde geçen "Context", yani `DbContext`.

Bunu "Veritabanı ile açılan güvenli bir tünel" olarak düşün. **Kritik Uyarı (Thread Safety):** `DbContext` **Thread-Safe DEĞİLDİR.** Aynı anda (paralel) iki farklı thread üzerinden aynı `context` nesnesini kullanarak işlem yapmaya çalışırsan uygulama patlar. Bu yüzden Web projelerinde her istek (Request) için yeni bir Context yaratılır ve iş bitince çöpe atılır (Scoped Lifetime).

###  Queries (Sorgular) - LINQ to Entities

"Queries" kısmı. EF Core, yazdığın C# kodunu (LINQ) alır ve SQL'e çevirir.

**Deferred Execution (Ertelenmiş Çalışma):** Bu kavramı anlamak hayati önem taşır.

C#

```csharp
var sorgu = context.Users.Where(u => u.Age > 18); // SQL gitmedi. Sadece tanım yapıldı.
var sonuc = sorgu.ToList(); // !!! SQL ŞİMDİ GİTTİ !!!
```

Sorguyu tanımlamak bedavadır, çalıştırmak (`ToList`, `Count`, `FirstOrDefault`) maliyetlidir. Bir döngü içinde yanlışlıkla sorguyu çalıştırırsan (Iteration), performansın çöker.

### Caching (Önbellekleme)

Metinde geçen "Caching". EF Core'da iki seviye önbellek vardır:

1. **First Level Cache (L1 - Context Cache):**
    
    - Her `DbContext`'in kendi içindedir.
        
    - Aynı Context yaşam döngüsü içinde, aynı ID'li veriyi 2 kere istersen ( `Find(1)` ), ikinci seferde veritabanına gitmez, hafızadan verir.
        
    - _Web uygulamalarında etkisi azdır_ çünkü her istekte Context yeniden yaratılır ve silinir.
        
2. **Second Level Cache (L2 - Distributed Cache):**
    
    - EF Core'un içinde gömülü gelmez (Hibernate'te vardır).
        
    - Redis veya NCache gibi araçlarla **senin kurman gerekir.**
        
    - Tüm uygulama genelinde (Context'ler arası) veriyi saklar. Veritabanı trafiğini %90 azaltabilir.