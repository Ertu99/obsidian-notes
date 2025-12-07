
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




EF Core kullanan çoğu geliştirici sadece `context.SaveChanges()` der ve sihrin gerçekleşmesini bekler. Ama bir **Backend Mühendisi**, o sihrin arkasında dönen çarkları bilmek zorundadır. Özellikle performans optimizasyonu (Bulk işlemler) ve karmaşık güncelleme senaryoları (Disconnected Entities) için bu API'yi manuel yönetmen gerekecek.

---

### 1. Change Tracker Nedir? (Felsefe: Gözetim Kulesi)

`ChangeTracker`, `DbContext` sınıfının içinde yaşayan ve hafızadaki tüm nesnelerin (Entity) "o anki durumunu" gözetleyen bir birimdir.1

Şunu hayal et: Bir otoparkın (DbContext) güvenlik kulesindesin. Otoparka giren her arabayı (Entity) izliyorsun.

- Bu araba yeni mi girdi? (`Added`)
    
- Rengi mi değişti? (`Modified`)
    
- Çıkış mı yaptı? (`Deleted`)
    
- Yoksa öylece duruyor mu? (`Unchanged`)
    

Sen `SaveChanges()` butonuna bastığında, güvenlik görevlisi (Change Tracker) elindeki bu listeye bakar ve veritabanına sadece gerekli komutları (`INSERT`, `UPDATE`, `DELETE`) gönderir.

---

### 2. Entity States (5 Temel Durum)

Change Tracker API'yi anlamak için, bir nesnenin alabileceği 5 durumu adın gibi bilmelisin. Her nesne herhangi bir anda bu durumlardan sadece birinde olabilir.

1. **Detached (Kopuk):** Entity hafızada var (C# nesnesi olarak), ama `DbContext` bundan haberdar değil. Gözetim altında değil. Veritabanına kaydedilmez.
    
    - _Örnek:_ `var user = new User();` (Henüz `Add` demedin).
        
2. **Unchanged (Değişmemiş):** Veritabanından çekildi ve üzerinde hiçbir değişiklik yapılmadı. `SaveChanges` çağrılırsa hiçbir şey olmaz.
    
3. **Added (Eklenmiş):** Context'e yeni eklendi (`.Add()`).2 Henüz veritabanında yok. `SaveChanges` sırasında `INSERT` sorgusu üretilir.3
    
4. **Modified (Değiştirilmiş):** Veritabanından geldi ama en az bir özelliği (Property) değişti. `SaveChanges` sırasında `UPDATE` sorgusu üretilir.4
    
5. **Deleted (Silinmiş):** Context üzerinden silindi (`.Remove()`).5 Veritabanında hala var ama `SaveChanges` sırasında `DELETE` sorgusu ile silinecek.
    

---

### 3. API'ye Erişim ve Manuel Müdahale

Çoğu zaman EF Core durumları otomatik yönetir. Ama bazen senin müdahale etmen gerekir.

**Erişim Yöntemleri:**

- `context.ChangeTracker`: Tüm izlenen nesnelere erişim sağlar.
    
- `context.Entry(entity)`: Tek bir nesnenin durumuna erişim sağlar.6
    

Mühendislik Örneği (Disconnected Scenario):

Bir API yazdın. Kullanıcı sana bir JSON gönderdi (User ID: 5, Name: "Yeni İsim"). Bu nesne senin hafızanda yeni (new User) olduğu için durumu Detached. Ama sen bunun veritabanında var olduğunu ve güncellenmesi gerektiğini biliyorsun. EF Core'a bunu nasıl anlatırsın?

C#

```csharp
public void UpdateUser(User user)
{
    // YÖNTEM 1: Klasik (Pahalı)
    // Önce DB'den çek, sonra değiştir. (2 Sorgu: SELECT + UPDATE)
    var existing = context.Users.Find(user.Id);
    existing.Name = user.Name;
    context.SaveChanges();

    // YÖNTEM 2: Change Tracker API (Performanslı)
    // DB'den çekmeye gerek yok! (Tek Sorgu: UPDATE)
    context.Attach(user); // Durumu 'Unchanged' yap
    context.Entry(user).State = EntityState.Modified; // Durumu elle 'Modified' yap
    context.SaveChanges();
}
```

Yöntem 2'de, veritabanına gitmeden (SELECT atmadan) nesneyi "sanki oradan gelmiş ve değişmiş gibi" sisteme tanıttık. Bu, Change Tracker API'nin gücüdür.

---

### 4. `Update` vs Property Modification (Kritik Fark)

Mülakatlarda çok sorulan bir detaydır.

**Senaryo:** Kullanıcı sadece "Email" adresini değiştirdi.

- **Yaklaşım A:** `user.Email = "yeni@mail.com";`
    
    - Change Tracker, sadece `Email` alanının değiştiğini fark eder.
        
    - Oluşan SQL: `UPDATE Users SET Email = '...' WHERE Id = 5`
        
    - _Sonuç:_ Optimize ve doğru.
        
- **Yaklaşım B:** `context.Users.Update(user);`
    
    - `Update` metodu, nesnenin **tüm alanlarını** `Modified` olarak işaretler.
        
    - Oluşan SQL: `UPDATE Users SET Name='...', Email='...', Age=... WHERE Id = 5`
        
    - _Sonuç:_ Gereksiz yere tüm sütunları günceller. Eğer o sırada başka biri "Age" alanını değiştirmişse, sen `Update` ile onun değişikliğini (eski veriyle) ezersin (**Last Write Wins** sorunu).
        

**Ders:** `context.Update()` metodunu, sadece tüm nesneyi gerçekten değiştirmek istiyorsan kullan. Aksi takdirde property bazlı değişiklik yap.

---

### 5. Debugging: `ChangeTracker.DebugView`

Kodun çalışıyor ama `SaveChanges` beklediğin şeyi yapmıyor mu? EF Core 5.0 ile gelen muazzam bir özellik var. Kodunu debug ederken `context.ChangeTracker.DebugView.LongView` özelliğine bakabilirsin.

Sana şöyle bir rapor verir:

Plaintext

```
User {Id: 1} Unchanged
  Id: 1 PK
  Name: 'Ahmet' (Modified) OriginalValue: 'Mehmet'
  Email: 'a@b.com'
Order {Id: 10} Added
  ...
```

Hangi nesne hangi durumda, hangi alan değişmiş, eski değeri neymiş; hepsini röntgen çeker gibi görürsün.

---

### 6. Performans: `AutoDetectChangesEnabled`

EF Core, sen her `context.Users.Add(user)` dediğinde veya `SaveChanges` çağırdığında, hafızadaki binlerce nesneyi tarayıp "Değişen var mı?" diye kontrol eder (`DetectChanges`).

Senaryo: 10.000 adet kayıt ekleyeceksin (Bulk Insert).

Döngü içinde 10.000 kere Add dersen, EF Core her seferinde DetectChanges çalıştırır. İşlem süresi karesel (Exponential) artar.

**Çözüm:**

C#

```csharp
// Gözetlemeyi geçici olarak kapat
context.ChangeTracker.AutoDetectChangesEnabled = false;

foreach(var user in users)
{
    context.Users.Add(user); // Artık çok hızlı, kontrol yapmıyor
}

// En sonda bir kere manuel çalıştır
context.ChangeTracker.DetectChanges();
context.SaveChanges();
```

Bu yöntemle 1 dakikalık işlemi 1 saniyeye indirebilirsin.

---

Özellikle metinde geçen **"Lazy Loading varsayılan davranıştır"** cümlesine dikkat! Bu, eski **Entity Framework 6 (.NET Framework)** için doğruydu. Ancak senin öğrendiğin modern **EF Core**'da varsayılan davranış **"Hiçbir Şeyi Yükleme" (null)** dir. Lazy Loading'i açmak için ekstra paket ve konfigürasyon gerekir. Bu hayati farkı bilmezsen sürekli `NullReferenceException` alırsın.

Bu üç stratejiyi, arka planda oluşturdukları SQL sorguları ve **Trade-off (Ödünleşim)** analizleriyle inceleyelim.

---

### 1. Eager Loading (Hırslı Yükleme) - "Peşin Peşin Al"

Veritabanına gitmişken "Ana veriyi alırken ilişkili verileri de (Çocukları) yanına kat getir" demektir.

- **Nasıl Yapılır?** `Include()` ve `ThenInclude()` metotları ile.
    
- **SQL Karşılığı:** `INNER JOIN` veya `LEFT JOIN`.
    
- **Avantajı:** Tek bir veritabanı turu (Roundtrip). Ağ trafiği için harikadır.
    

Mühendislik Riski: Cartesian Explosion (Kartezyen Patlaması)

Eager Loading masum görünür ama tehlikelidir.

Senaryo: 100 Kullanıcı, her birinin 50 Siparişi, her siparişin 10 Detayı var.

context.Users.Include(u => u.Orders).ThenInclude(o => o.Details).ToList()

- SQL Server sana `100 x 50 x 10 = 50.000` satırlık devasa bir tablo döndürür.
    
- Her satırda Kullanıcı bilgileri (Ad, Soyad) 500 kere tekrar eder.
    
- **Sonuç:** Ağ tıkanır, RAM şişer.
    

**Çözüm (Split Queries):** EF Core 5.0 ile gelen `.AsSplitQuery()` metodu. "Tek bir dev JOIN atma, önce kullanıcıları çek, sonra siparişleri çek, bellekte birleştir" der.

---

### 2. Lazy Loading (Tembel Yükleme) - "Lazım Olursa Al"

Ana veriyi çekersin. İlişkili veriye (`user.Orders`) kod içinde eriştiğin **o anda** veritabanına gidip çeker.

- **EF Core'da Kurulum (Default Değildir!):**
    
    1. `Microsoft.EntityFrameworkCore.Proxies` paketini yükle.
        
    2. `DbContext` ayarlarında `.UseLazyLoadingProxies()` de.
        
    3. Entity sınıflarındaki ilişkili property'lere **`virtual`** keyword'ünü ekle (`public virtual ICollection<Order> Orders { get; set; }`).
        
- **Nasıl Çalışır?** EF Core, senin sınıfından miras alan dinamik bir sınıf (Proxy) üretir. Sen `Orders` property'sine dokunduğunda (get), araya giren kod veritabanına sorgu atar.
    

Mühendislik Riski: N+1 Problemi

Bunu daha önce konuşmuştuk ama tekrar edelim çünkü Lazy Loading'in ölümcül günahıdır.

C#

```csharp
var users = context.Users.ToList(); // 1 Sorgu (SELECT * FROM Users)

foreach (var user in users) 
{
    // DÖNGÜ İÇİNDE SQL ÇAĞRISI!
    Console.WriteLine(user.Orders.Count); 
}
```

1000 kullanıcı varsa, 1001 sorgu atılır. Veritabanı CPU'su tavan yapar.

---

### 3. Explicit Loading (Açık Yükleme) - "Manuel Kontrol"

Ne Eager gibi baştan alır, ne Lazy gibi gizlice arkadan alır. Verinin ne zaman yükleneceğine tamamen **sen karar verirsin.**

- **Nasıl Yapılır?** `context.Entry(...)` API'si ile.
    
- **Kullanım Alanı:** Koşullu yükleme.
    

C#

```csharp
var user = context.Users.Find(1); // Sadece User geldi

if (user.IsVip) 
{
    // Sadece VIP ise siparişlerini yükle
    context.Entry(user)
           .Collection(u => u.Orders)
           .Load(); 
}
```

- **Query ile Filtreleme:** Explicit loading'in en büyük gücü, yüklerken filtreleme yapabilmesidir.
    
    C#
    
    ```csharp
    context.Entry(user)
           .Collection(u => u.Orders)
           .Query() // Henüz yükleme, sorguya devam et
           .Where(o => o.Price > 1000) // Sadece 1000 TL üzeri siparişleri getir
           .Load();
    ```
    
    Bu özelliği `Include` (Eager) ile de yapabilirsin (EF Core 5+) ama Explicit Loading daha granüler kontrol sağlar.
    

---

 EF Core'un final konusu ve modern yazılım geliştirmede en sık kullanılan, en çok hata yapılan özellik: **Code First Migrations**.

ORM kullanmanın en büyük avantajı, veritabanı şemasını (Tabloları, Sütunları) SQL yazarak değil, C# class'larını değiştirerek yönetmektir. Ancak bu güç, büyük sorumluluk getirir. Yanlış bir migration, Production ortamındaki **tüm veriyi silebilir.**

Bu konuyu sadece "komut yaz çalıştır" olarak değil, **Schema Versioning (Şema Versiyonlama)** stratejisi ve **CI/CD Pipeline** uyumu üzerinden mühendislik derinliğinde inceleyeceğiz.

---

### 1. Felsefe: Veritabanını Git Gibi Yönetmek

Eskiden (Database First), veritabanında bir değişiklik yapıldığında (örn: `Users` tablosuna `Age` ekle), bu değişikliği diğer geliştiricilere veya Canlı ortama (Production) taşımak için `.sql` dosyaları elden ele dolaşırdı.

**Code First Migrations**, veritabanının versiyon kontrol sistemidir.

- **Snapshot (Anlık Görüntü):** Veritabanının şu anki halini bilir.
    
- **Diff (Fark):** Senin C# kodunda yaptığın değişikliği algılar.
    
- **Patch (Yama):** Aradaki farkı kapatacak SQL kodunu (`ALTER TABLE...`) otomatik üretir.
    

---

### 2. Mekanizma: Nasıl Çalışır? (Kaputun Altı)

Sen `Add-Migration` komutunu çalıştırdığında EF Core, C# kodunu (Entity) veritabanı ile karşılaştırmaz. **Burası çok yanlış bilinir.**

EF Core şu üçlüyü karşılaştırır:

1. **C# Entity Class'ları:** Senin son yazdığın kod.
    
2. **`DbContextModelSnapshot.cs`:** `Migrations` klasöründe duran, veritabanının (bildiği kadarıyla) son hali.
    
3. **Veritabanı (`__EFMigrationsHistory` Tablosu):** Hangi migration'ların uygulandığını tutan özel tablo.
    

Eğer C# kodunda `User` sınıfına `string Phone` eklediysen; EF Core, Snapshot'a bakar, "Bende Phone yoktu" der ve aradaki fark için bir Migration dosyası üretir.

---

### 3. Bir Migration Dosyasının Anatomisi (`Up` vs `Down`)

Oluşturulan her migration sınıfında iki kritik metot vardır:

1. **`Up()` (İleri Git):** Değişikliği uygular.
    
    - _Örnek:_ `CreateTable(...)`, `AddColumn(...)`
        
2. **`Down()` (Geri Al):** Yapılan değişikliği geri alır.
    
    - _Örnek:_ `DropTable(...)`, `DropColumn(...)`
        

**Mühendislik Kuralı:** `Down` metodu her zaman `Up` metodunun tam tersi olmalıdır. Böylece bir hata anında `Update-Database <ÖncekiMigration>` diyerek sistemi güvenle geri alabilirsin.

---

### 4. Kritik Komutlar ve Yaşam Döngüsü

Terminalde (veya Package Manager Console'da) en sık kullanacağın komutlar:

1. **`Add-Migration <Isim>`:**
    
    - C# kodundaki değişiklikleri algılar ve yeni bir `.cs` migration dosyası oluşturur. Veritabanına dokunmaz.
        
    - _İsimlendirme Önemlidir:_ `AddColumnAgeToUsers` gibi açıklayıcı isimler ver. `Migration1`, `Deneme` gibi isimler verme.
        
2. **`Update-Database`:**
    
    - Bekleyen migration'ları (`__EFMigrationsHistory` tablosuna bakarak) bulur ve veritabanına uygular.
        
3. **`Script-Migration` (Profesyonel Yöntem):**
    
    - Migration'ı doğrudan uygulamak yerine, çalıştıracağı **SQL kodunu** sana verir.
        
    - _Neden?_ DBA (Veritabanı Yöneticisi), kodu Production'a atmadan önce incelemek isteyebilir.
        
4. **`Remove-Migration`:**
    
    - Henüz veritabanına **uygulanmamış** son migration'ı siler. Hatalı oluşturduysan kullanırsın.
        

---

### 5. Mühendislik Tuzağı: Veri Kaybı (Data Loss)

Bu konuyu "derinlemesine" istediğin için en büyük riski anlatıyorum: **Rename (Yeniden İsimlendirme).**

**Senaryo:** `Users` tablosundaki `FullName` sütununun adını `Name` olarak değiştirmek istedin.

1. C# kodunda `FullName`'i sildin, `Name` yazdın.
    
2. `Add-Migration` dedin.
    

**EF Core Ne Görür?**

- "Hımm, `FullName` diye bir şey yok olmuş -> `DropColumn(FullName)`"
    
- "Hımm, `Name` diye yeni bir şey gelmiş -> `AddColumn(Name)`"
    

**Sonuç:** `Update-Database` dediğin an, `FullName` sütunu içindeki **tüm verilerle birlikte silinir.** Yeni boş bir `Name` sütunu açılır. Veri kaybı yaşanır.

**Çözüm:** Migration dosyasını açıp manuel müdahale etmelisin: `migrationBuilder.RenameColumn(name: "FullName", newName: "Name")`.

---

### 6. Production Stratejisi: `Migrate()` on Startup?

Yazılımcılar genelde `Program.cs` içine şunu yazar:

C#

```csharp
// Uygulama başlarken otomatik migrate et
using (var scope = app.Services.CreateScope()) {
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate(); 
}
```

**Mühendislik Uyarısı:** Bu kod küçük projelerde harikadır. Ama **High-Scale (Yüksek Trafikli)** ve **Cluster (Küme)** ortamlarda (Kubernetes gibi) felakettir.

- Neden? Aynı anda 5 tane sunucu (Pod) ayağa kalkarsa, 5'i birden aynı anda veritabanı şemasını değiştirmeye çalışır (**Concurrency Conflict**). Veritabanı kilitlenir (Deadlock).
    
- **Doğrusu:** CI/CD pipeline içinde (GitHub Actions vb.) `Script-Migration` ile SQL üretip, deploy aşamasında tek bir yetkili servisin bunu çalıştırmasıdır.
    

---

