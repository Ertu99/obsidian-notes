# Yüksek Performanslı.NET Core Mikroservis Mimarilerinde Dapper: Kapsamlı Uygulama, Optimizasyon ve Mimari Entegrasyon Raporu

## 1. Yönetici Özeti ve Teorik Temeller

Modern yazılım mühendisliği ekosisteminde, özellikle dağıtık mikroservis mimarilerinin yükselişiyle birlikte, veri erişim katmanlarının (Data Access Layer - DAL) verimliliği, genel sistem performansının belirleyici faktörü haline gelmiştir. Geleneksel Object-Relational Mapping (ORM) araçları, geliştirici deneyimini iyileştirmek adına sundukları yüksek soyutlama seviyeleri nedeniyle, milisaniyelerin kritik olduğu senaryolarda darboğaz oluşturabilmektedir. Bu bağlamda, "Micro-ORM" sınıfının lideri olarak kabul edilen Dapper, ham SQL gücünü nesne yönelimli programlamanın konforuyla birleştiren stratejik bir çözüm olarak öne çıkmaktadır.

Bu rapor, kurumsal ölçekli.NET Core mikroservis projelerinde Dapper'ın entegrasyonunu, konfigürasyonunu ve ileri seviye kullanım senaryolarını, teorik derinlik ve pratik uygulama detaylarıyla birlikte ele almaktadır. Raporun temel amacı, Entity Framework Core (EF Core) gibi tam kapsamlı ORM'lerden geçiş yapan yazılım mühendislerine, Dapper'ın iç mekanizmalarını, bellek yönetimini ve mimari desenlerle uyumunu en ince detayına kadar aktarmaktır.

### 1.1 ORM Paradigması ve "Impedance Mismatch" Sorunu

Nesne Yönelimli Programlama (OOP) ile İlişkisel Veritabanı Yönetim Sistemleri (RDBMS) arasındaki yapısal uyumsuzluk, yazılım dünyasında "Object-Relational Impedance Mismatch" olarak adlandırılır. Geleneksel ORM'ler (EF Core, NHibernate), bu uyumsuzluğu gidermek için veritabanı tablolarını bellekteki nesnelere eşler ve durum takibi (change tracking) yapar. Ancak bu süreç, "Unit of Work" deseninin getirdiği ağır bellek yükünü ve çalışma zamanında dinamik SQL oluşturma maliyetini beraberinde getirir.

Dapper, bu soruna radikal bir yaklaşım getirerek "İlişkisel Yönetim" yerine saf "Nesne Eşleme"ye odaklanır. Stack Overflow ekibi tarafından, platformun yüksek trafik yükünü karşılamak amacıyla geliştirilen Dapper, `IDbConnection` arayüzünü genişleterek ADO.NET'in ham performansına en yakın deneyimi sunar. Dapper, LINQ sorgularını SQL'e çevirmek veya nesnelerin durumunu takip etmek yerine, doğrudan yazılan SQL sorgularının sonuçlarını POCO (Plain Old CLR Objects) sınıflarına eşler. Bu yöntem, EF Core'un getirdiği karmaşıklık katmanlarını ortadan kaldırarak, sorgu işleme sürelerinde 5 ila 10 kat arasında performans artışı sağlar.1

**Tablo 1: Dapper ve Entity Framework Core Arasındaki Mimari ve Performans Karşılaştırması**

|**Özellik**|**Dapper**|**Entity Framework Core**|
|---|---|---|
|**Soyutlama Seviyesi**|Düşük (Micro-ORM); metale yakın.|Yüksek (Full ORM); SQL üretimini soyutlar.|
|**Sorgu Mekanizması**|Ham SQL veya Saklı Yordamlar (Stored Procedures).|LINQ (Language Integrated Query).|
|**Durum Takibi (Change Tracking)**|Yok (Stateless - Durumsuz).|Otomatik (Stateful - Durumlu Context).|
|**Bellek Kullanımı (Tekil Ekleme)**|Düşük (~18.23 KB).2|Yüksek (~39.09 KB).2|
|**Toplu Ekleme (Bulk Insert - 30 Kayıt)**|Verimli (~427.73 KB).2|Daha Yüksek (~753.61 KB).2|
|**Kullanım Senaryosu**|Yüksek frekanslı okumalar, karmaşık SQL, CQRS Read Stack.|Karmaşık iş kuralları, hızlı prototipleme, Graph Save.|

### 1.2 Mikroservis Mimarilerinde Dapper'ın Rolü

Mikroservis mimarisi, servislerin otonom olmasını ve kendi veri tabanlarını yönetmesini gerektirir. Dağıtık bir sistemde, bir servisteki gecikme zincirleme reaksiyonla tüm sistemi etkileyebilir. Dapper, bu mimaride aşağıdaki kritik rolleri üstlenir:

1. **CQRS (Command Query Responsibility Segregation):** Modern mimarilerde okuma ve yazma modellerinin ayrılması yaygındır. Dapper, özellikle "Okuma" (Query) tarafında, veriyi en hızlı şekilde istemciye ulaştırmak için idealdir. Karmaşık join işlemleri ve optimize edilmiş SQL sorguları, Dapper üzerinden EF Core'a göre çok daha verimli çalıştırılır.3
    
2. **Legacy (Eski) Sistem Entegrasyonu:** Mikroservisler bazen modern olmayan, karmaşık saklı yordamlar (stored procedures) içeren veritabanlarıyla konuşmak zorunda kalabilir. EF Core'un standartlarına uymayan bu yapılarda, Dapper'ın esnek parametre yönetimi ve ham SQL desteği hayati önem taşır.4
    
3. **Kaynak Optimizasyonu:** Dapper'ın düşük bellek ayak izi (memory footprint), konteyner (Docker/Kubernetes) ortamlarında çalışan mikroservislerin daha az RAM tüketerek ölçeklenmesine (scaling) olanak tanır. Yapılan testlerde, toplu veri işlemlerinde Dapper'ın bellek tüketiminin EF Core'a kıyasla %40 daha az olduğu gözlemlenmiştir.2
    

---

## 2..NET Core Ortamında Kurulum ve Konfigürasyon Stratejileri

Dapper'ın bir mikroservis projesine entegrasyonu, sadece bir NuGet paketi yüklemekten ibaret değildir. Doğru bir başlangıç, `IDbConnection` nesnesinin yaşam döngüsünü (lifecycle) ve Bağımlılık Enjeksiyonu (Dependency Injection - DI) konteynerindeki yapılandırmasını doğru yönetmeyi gerektirir.

### 2.1 Paket Yönetimi ve Gerekli Kütüphaneler

Dapper, `System.Data` isim uzayı üzerine kurulu bir genişletme kütüphanesidir. Projenize dahil etmek için NuGet Package Manager veya.NET CLI kullanılabilir. SQL Server tabanlı bir mikroservis için temel paketlerin yanı sıra, Dapper'ın yeteneklerini artıran yan paketlerin de kurulması önerilir 5:

- `Dapper`: Çekirdek kütüphane. `Query`, `Execute` gibi genişletme metodlarını barındırır.
    
- `Microsoft.Data.SqlClient`:.NET Core için optimize edilmiş SQL Server veri sağlayıcısıdır (`System.Data.SqlClient` yerine tercih edilmelidir).
    
- `Dapper.Contrib`: CRUD operasyonlarını otomatize etmek için kullanılan resmi eklenti paketi.7
    

PostgreSQL gibi farklı veritabanları kullanılıyorsa, `Npgsql` gibi ilgili sağlayıcıların projeye eklenmesi gerekir.8

### 2.2 Bağımlılık Enjeksiyonu (DI) ve Bağlantı Yaşam Döngüsü

.NET Core mimarisinde yapılan en yaygın hatalardan biri, veritabanı bağlantısının `Singleton` olarak kaydedilmesidir. Veritabanı bağlantıları "thread-safe" (iş parçacığı güvenli) değildir. Tek bir bağlantının birden fazla istek tarafından aynı anda kullanılması, veri karışıklığına ve yarış durumlarına (race conditions) yol açar.

Önerilen Desen: Bağlantı Fabrikası (Connection Factory) veya Context Wrapper

En güvenilir yaklaşım, bağlantıyı doğrudan enjekte etmek yerine, bağlantıyı oluşturacak bir fabrika veya bağlam (context) sınıfı tasarlamaktır. Bu sınıf Singleton olabilir, ancak ürettiği bağlantı Transient (geçici) veya Scoped (istek bazlı) yaşam döngüsüne sahip olmalıdır.9 ADO.NET'in kendi içinde barındırdığı "Connection Pooling" mekanizması, bağlantıların fiziksel olarak sürekli açılıp kapanmasını engeller; ancak yazılımsal nesnenin (SqlConnection) işi bittiğinde mutlaka Dispose edilmesi gerekir.

Aşağıdaki `DapperContext` sınıfı, konfigürasyon dosyasından (`appsettings.json`) bağlantı dizesini okuyarak ihtiyaç anında yeni bir bağlantı nesnesi üretir 5:

C#

```
public class DapperContext
{
    private readonly IConfiguration _configuration;
    private readonly string _connectionString;

    public DapperContext(IConfiguration configuration)
    {
        _configuration = configuration;
        _connectionString = _configuration.GetConnectionString("DefaultConnection");
    }

    // Her çağrıldığında yeni bir IDbConnection örneği döner.
    // Bu nesne kullanıldıktan sonra mutlaka Dispose edilmelidir (using bloğu ile).
    public IDbConnection CreateConnection()
        => new SqlConnection(_connectionString);
}
```

`Program.cs` dosyasında bu servis şu şekilde kaydedilir:

C#

```
builder.Services.AddSingleton<DapperContext>();
```

Bu yapı, servisin herhangi bir katmanında `DapperContext` enjekte edilerek güvenli bir şekilde veritabanı bağlantısı açılmasını sağlar. Bu yaklaşım, bağlantı yönetimini merkezi bir noktada tutarken, her HTTP isteğinin kendi izole bağlantı alanında çalışmasını garanti eder.

---

## 3. Çekirdek Veri Erişim Operasyonları (CRUD)

Dapper'ın gücü, `IDbConnection` arayüzüne eklediği genişletme metodlarında yatar. Bu metodlar, ham SQL sorgularını çalıştırıp sonuçları.NET nesnelerine dönüştürür. Mikroservislerin yüksek trafik altında bloklanmadan çalışabilmesi için **asenkron (Async)** metodların kullanımı zorunludur.6

### 3.1 Veri Okuma: Query Metod Ailesi

`Query<T>` ve türevleri, veritabanından veri çekmek (SELECT) için kullanılır. Dapper, veritabanından dönen sütun isimleri ile `T` tipindeki sınıfın özellik (property) isimlerini eşleştirir. Bu eşleşme varsayılan olarak büyük/küçük harf duyarsızdır.

#### 3.1.1 Liste Sorgulama (GetAll)

Birden fazla kaydın çekildiği senaryolarda `QueryAsync` metodu `IEnumerable<T>` döner.

C#

```
public async Task<IEnumerable<Urun>> TumUrunleriGetirAsync()
{
    var sql = "SELECT Id, Ad, Fiyat FROM Urunler";
    
    // using bloğu, bağlantının iş bittiğinde havuza iade edilmesini (dispose) garanti eder.
    using (var connection = _context.CreateConnection())
    {
        // QueryAsync metodu veriyi asenkron olarak çeker ve Urun nesnelerine map eder.
        var urunler = await connection.QueryAsync<Urun>(sql);
        return urunler.ToList();
    }
}
```

#### 3.1.2 Tekil Kayıt Sorgulama

Tek bir satırın beklendiği durumlarda Dapper, LINQ benzeri semantikler sunar. Hangi metodun seçileceği, verinin varlığına ve benzersizliğine dair iş kurallarına bağlıdır 6:

- **`QueryFirstAsync<T>`**: Sonuç kümesindeki ilk satırı döner. Eğer birden fazla satır varsa hata vermez, diğerlerini görmezden gelir. Hiç satır yoksa hata fırlatır.
    
- **`QueryFirstOrDefaultAsync<T>`**: İlk satırı döner; eğer hiç satır yoksa `null` (veya default değer) döner. Genellikle "ID ile bul" senaryolarında en güvenli yöntemdir.
    
- **`QuerySingleAsync<T>`**: Kesinlikle tek bir satır bekler. Hiç satır yoksa veya birden fazla satır varsa hata fırlatır. Veri bütünlüğünün kritik olduğu durumlarda kullanılır.
    
- **`QuerySingleOrDefaultAsync<T>`**: Sıfır veya bir satır bekler. Birden fazla satır varsa hata fırlatır.
    

### 3.2 Veri Değiştirme: Execute Metodu

INSERT, UPDATE ve DELETE işlemleri (Data Manipulation Language - DML) için `ExecuteAsync` metodu kullanılır. Bu metod, işlemden etkilenen satır sayısını (`int`) döner.6

C#

```
public async Task<int> UrunGuncelleAsync(Urun urun)
{
    var sql = "UPDATE Urunler SET Ad = @Ad, Fiyat = @Fiyat WHERE Id = @Id";
    using (var connection = _context.CreateConnection())
    {
        // Dapper, 'urun' nesnesinin özelliklerini SQL parametrelerine (@Ad, @Fiyat) otomatik eşler.
        return await connection.ExecuteAsync(sql, urun);
    }
}
```

Bu örnekte Dapper'ın **Parametre Enjeksiyonu** özelliği görülmektedir. SQL string'i içine doğrudan değer yazmak yerine `@Parametre` notasyonu kullanmak, uygulamanızı **SQL Injection** saldırılarına karşı korur. Dapper, gönderilen nesnenin property'lerini analiz eder ve parametreleri güvenli bir şekilde SQL komutuna ekler.

### 3.3 Ekleme İşlemi ve ID Geri Dönüşü (Insert & Return ID)

İlişkisel veritabanlarında yeni bir kayıt eklendiğinde, genellikle veritabanı tarafından üretilen otomatik artan ID'ye (Primary Key) ihtiyaç duyulur. `ExecuteAsync` sadece etkilenen satır sayısını döndüğü için, ID'yi almak adına sorguya bir `SELECT` ifadesi eklenmeli ve `QuerySingleAsync` kullanılmalıdır.12

C#

```
// SQL Server için SCOPE_IDENTITY() kullanımı
var sql = @"INSERT INTO Urunler (Ad, Fiyat) VALUES (@Ad, @Fiyat); 
            SELECT CAST(SCOPE_IDENTITY() as int);";

using (var connection = _context.CreateConnection())
{
    // Execute yerine QuerySingle kullanarak dönen ID'yi yakalıyoruz.
    var yeniId = await connection.QuerySingleAsync<int>(sql, urun);
    return yeniId;
}
```

SQL Server'ın `OUTPUT` komutu da benzer bir amaçla kullanılabilir ve tek bir round-trip (sunucuya gidiş-dönüş) ile eklenen tüm satır verisini geri döndürebilir.

---

## 4. İleri Seviye Parametre Yönetimi ve Stored Procedure Entegrasyonu

Gerçek hayat senaryolarında, basit nesne-parametre eşleşmesi her zaman yeterli olmaz. Saklı yordamlar (Stored Procedures), dinamik filtrelemeler ve veritabanı spesifik tipler, daha gelişmiş bir parametre yönetimini zorunlu kılar.

### 4.1 DynamicParameters Kullanımı

`DynamicParameters` sınıfı, Dapper'ın parametre çantası (parameter bag) mekanizmasıdır. Bu yapı, parametrelerin çalışma zamanında dinamik olarak oluşturulmasına, veritabanı tiplerinin (`DbType`) ve parametre yönlerinin (`Input`, `Output`, `ReturnValue`) açıkça belirtilmesine olanak tanır.13

Özellikle eski sistemlerle (Legacy) çalışırken, geriye değer döndüren saklı yordamlar yaygındır.

C#

```
var parametreler = new DynamicParameters();
parametreler.Add("@KullaniciId", 101);
parametreler.Add("@ToplamTutar", dbType: DbType.Decimal, direction: ParameterDirection.Output);
parametreler.Add("@IslemSonucu", dbType: DbType.Int32, direction: ParameterDirection.ReturnValue);

await connection.ExecuteAsync("SiparisHesapla", parametreler, commandType: CommandType.StoredProcedure);

// İşlem bittikten sonra çıktı parametrelerine erişim
var toplam = parametreler.Get<decimal>("@ToplamTutar");
var sonuc = parametreler.Get<int>("@IslemSonucu");
```

Bu yöntem, Dapper'ın sadece basit SQL sorguları için değil, karmaşık veritabanı mantığı içeren senaryolar için de ne kadar yetkin olduğunu göstermektedir.15

### 4.2 Liste Parametreleri (WHERE IN Clause)

Bir e-ticaret uygulamasında "Sepetteki şu ürünlerin detaylarını getir" gibi bir istekte, SQL'in `WHERE IN` yapısına ihtiyaç duyulur. Geleneksel ADO.NET'te bu işlem için dinamik string birleştirme yapmak gerekirken, Dapper `IEnumerable` tipindeki bir listeyi otomatik olarak parametrelere ayırır (Expansion).14

C#

```
var urunIdleri = new { 1, 5, 12, 20 };
var sql = "SELECT * FROM Urunler WHERE Id IN @Idler";

// Dapper bunu arka planda şuna çevirir:... WHERE Id IN (@Idler1, @Idler2, @Idler3...)
var urunler = await connection.QueryAsync<Urun>(sql, new { Idler = urunIdleri });
```

Bu özellik, geliştiriciyi karmaşık döngülerden kurtarır ve sorgu performansını optimize eder.18

### 4.3 Table-Valued Parameters (TVP)

Eğer binlerce kaydın aynı anda filtrelenmesi veya eklenmesi gerekiyorsa, `WHERE IN` operatörü performans sorunlarına yol açabilir (SQL Server'da parametre sınırı vardır). Bu gibi yüksek hacimli veri transferlerinde, Dapper **Table-Valued Parameters (TVP)** desteği sunar. `DataTable` veya `IEnumerable<SqlDataRecord>` kullanılarak, veriler yapısal bir tablo formunda veritabanına tek seferde gönderilebilir. Bu, özellikle "Bulk Insert" veya karmaşık raporlama işlemlerinde mikroservis performansını ciddi oranda artırır.20

---

## 5. Karmaşık Nesne Eşleme ve İlişkisel Veri Yönetimi

Mikroservisler genellikle "Domain Driven Design" (DDD) prensiplerini uygular ve bu da zengin domain modelleri (Aggregate Root) anlamına gelir. Örneğin, bir `Sipariş` nesnesi içinde `Musteri` bilgisi ve `SiparisKalemleri` listesi bulunabilir. Dapper, düz (flat) SQL sonuçlarını bu hiyerarşik nesne yapılarına dönüştürmek için güçlü mekanizmalar sunar.

### 5.1 One-to-One (Bire-Bir) Eşleme

Bir SQL JOIN işlemi sonucunda, hem `Siparis` hem de `Musteri` bilgilerini içeren geniş bir tablo döner. Dapper'ın "Multi-Mapping" özelliği, bu tek satırı bölerek birden fazla nesneye dağıtır.21

C#

```
var sql = @"SELECT s.Id, s.Tarih, m.Id, m.Ad 
            FROM Siparisler s 
            INNER JOIN Musteriler m ON s.MusteriId = m.Id";

// <Siparis, Musteri, Siparis> -> İlk tip, İkinci tip, Dönüş tipi
var siparisler = await connection.QueryAsync<Siparis, Musteri, Siparis>(
    sql,
    (siparis, musteri) => 
    {
        siparis.Musteri = musteri; // Nesneleri birbirine bağlıyoruz
        return siparis;
    },
    splitOn: "Id" // Dapper'a veriyi nereden böleceğini söylüyoruz
);
```

`splitOn` argümanı varsayılan olarak "Id"dir. Ancak, birleşen tabloların anahtar sütunları farklı isimlendirilmişse (örneğin `MusteriKodu`), bu parametrenin açıkça belirtilmesi gerekir. Dapper, sonuç kümesini okurken `splitOn` ile belirtilen sütunu gördüğü anda, o noktadan sonrasının ikinci nesneye ait olduğunu anlar.21

### 5.2 One-to-Many (Bire-Çok) Eşleme

Bir siparişin birden fazla kalemi olduğunda (One-to-Many), SQL sorgusu sipariş bilgilerini tekrar eden satırlar halinde getirir. Dapper bu satırları otomatik olarak gruplamaz; bu işlemi geliştiricinin bir `Dictionary` (Sözlük) yapısı kullanarak yönetmesi gerekir.23

C#

```
var sql = @"SELECT s.Id, s.Tarih, k.Id, k.UrunAdi, k.Fiyat 
            FROM Siparisler s 
            INNER JOIN SiparisKalemleri k ON s.Id = k.SiparisId";

var siparisSozlugu = new Dictionary<int, Siparis>();

var liste = await connection.QueryAsync<Siparis, Kalem, Siparis>(
    sql,
    (siparis, kalem) =>
    {
        if (!siparisSozlugu.TryGetValue(siparis.Id, out var mevcutSiparis))
        {
            mevcutSiparis = siparis;
            mevcutSiparis.Kalemler = new List<Kalem>();
            siparisSozlugu.Add(mevcutSiparis.Id, mevcutSiparis);
        }
        mevcutSiparis.Kalemler.Add(kalem);
        return mevcutSiparis;
    },
    splitOn: "Id"
);

// Sözlükteki değerleri liste olarak dönüyoruz
return siparisSozlugu.Values;
```

Bu yöntem, EF Core'un "Lazy Loading" mekanizmasının yarattığı "N+1 Select" problemini ortadan kaldırır. Tüm veri tek bir sorgu ile çekilir ve bellekte işlenir.24

### 5.3 Multiple Result Sets (Çoklu Sonuç Kümeleri)

Performansın en üst düzeye çıkarılması gereken dashboard (gösterge paneli) gibi senaryolarda, birden fazla bağımsız sorguyu tek seferde veritabanına göndermek (Query Batching) ağ trafiğini azaltır. `QueryMultipleAsync` metodu, GridReader kullanarak sıralı sonuç kümelerini okumayı sağlar.11

C#

```
var sql = @"SELECT * FROM Kullanicilar WHERE Id = @Id; 
            SELECT * FROM Siparisler WHERE KullaniciId = @Id;";

using (var multi = await connection.QueryMultipleAsync(sql, new { Id = kullaniciId }))
{
    var kullanici = await multi.ReadFirstAsync<Kullanici>();
    var siparisler = (await multi.ReadAsync<Siparis>()).ToList();
    // Kullanici ve siparişleri tek bir DTO'da birleştirip dönebiliriz.
}
```

### 5.4 JSON Eşleme ve Custom Type Handlers

Günümüz veritabanlarında (SQL Server, PostgreSQL) JSON formatında veri saklamak yaygındır. Ancak Dapper, varsayılan olarak bir JSON string'ini karmaşık bir C# nesnesine (`List<string>` veya `DetayNesnesi`) dönüştüremez. Bunun için `SqlMapper.TypeHandler<T>` sınıfı devreye girer. `System.Text.Json` veya `Newtonsoft.Json` kullanılarak özel bir dönüştürücü yazılabilir.27

C#

```
public class JsonTypeHandler<T> : SqlMapper.TypeHandler<T>
{
    public override void SetValue(IDbDataParameter parameter, T value)
    {
        parameter.Value = JsonSerializer.Serialize(value);
    }

    public override T Parse(object value)
    {
        return JsonSerializer.Deserialize<T>(value as string);
    }
}

// Program.cs içinde kayıt
SqlMapper.AddTypeHandler(new JsonTypeHandler<KullaniciAyarlari>());
```

Bu sayede veritabanındaki JSON sütunları, C# kodunda tamamen şeffaf bir şekilde nesne olarak kullanılır.29

---

## 6. Performans Optimizasyonu ve Dahili Mekanizmalar

Dapper'ın hızının arkasındaki sır, **IL (Intermediate Language) Emitting** tekniğidir. Dapper, çalışma zamanında SQL sonucunu nesneye eşleyecek kodu dinamik olarak derler ve önbelleğe (cache) alır. Bu sayede, ikinci çağrıdan itibaren neredeyse elle yazılmış kod kadar hızlı çalışır.

### 6.1 Buffered vs. Unbuffered Sorgular

Dapper varsayılan olarak "Buffered" (Tamponlanmış) modda çalışır. Yani, sorgu sonucu dönen tüm satırlar önce belleğe (`List<T>`) alınır, veritabanı bağlantısı kapatılır ve sonra liste geliştiriciye verilir.

- **Buffered (Varsayılan):**
    
    - Avantajı: Veritabanı bağlantısı hızla serbest bırakılır.
        
    - Dezavantajı: Çok büyük veri setlerinde (örneğin 1 milyon satır) bellekte `OutOfMemoryException` hatasına yol açabilir.31
        
- **Unbuffered (`buffered: false`):**
    
    - Veriler `IEnumerable` olarak tek tek (streaming) okunur. Veri okundukça bellekten atılır.
        
    - Kritik Nokta: Veri okunurken veritabanı bağlantısı açık kalır. Eğer veriyi işleyen kod yavaşsa, bağlantı havuzu şişebilir ve sistem kilitlenebilir. Sadece çok büyük raporlama işlemlerinde kullanılmalıdır.32
        

C#

```
// Unbuffered kullanım
var veriler = await connection.QueryAsync<LogKaydi>(sql, buffered: false);
```

### 6.2 Performans Metrikleri

Yapılan benchmark testlerinde, Dapper'ın tekil veri ekleme (insert) işleminde EF Core'a göre **yarı yarıya daha az bellek** kullandığı (18 KB vs 39 KB) görülmüştür. Toplu işlemlerde bu fark daha da açılır. Ancak, EF Core'un son sürümleri (EF Core 8/9) aradaki farkı kapatmaya başlamıştır; yine de "Tracking" mekanizmasının olmadığı senaryolarda Dapper, işlemci döngülerini (CPU Cycles) daha verimli kullanır.1

---

## 7. İşlem Yönetimi (Transaction Management) ve Veri Bütünlüğü

Mikroservislerde veri tutarlılığı (Consistency) en zorlu konulardan biridir. Servis içi tutarlılık için ACID prensipleri uygulanmalıdır.

### 7.1 Manuel Transaction Yönetimi (IDbTransaction)

En temel yöntem, `IDbTransaction` nesnesini manuel yönetmektir. Burada dikkat edilmesi gereken en önemli kural, transaction başlatıldıktan sonra yapılan **her Dapper çağrısına** bu transaction nesnesinin parametre olarak geçilmesidir. Aksi takdirde sorgu transaction dışında çalışır ve bütünlük bozulur.34

C#

```
using (var connection = _context.CreateConnection())
{
    connection.Open();
    using (var transaction = connection.BeginTransaction())
    {
        try
        {
            await connection.ExecuteAsync(sql1, param, transaction: transaction);
            await connection.ExecuteAsync(sql2, param, transaction: transaction); // Transaction parametresi zorunlu!
            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```

### 7.2 TransactionScope ve Async Flow Tehlikesi

.NET'te `TransactionScope` bloğu kullanmak, transaction nesnesini her metoda parametre geçme zorunluluğunu ortadan kaldırır (Ambient Transaction). Ancak, `async/await` desenleri ile kullanıldığında,.NET Framework 4.5.1 öncesinden gelen bir hata nedeniyle transaction bağlamı (context) thread değiştirildiğinde kaybolabilir.

.NET Core'da `TransactionScope` kullanırken **mutlaka** `TransactionScopeAsyncFlowOption.Enabled` seçeneği aktif edilmelidir. Bu yapılmazsa, `await` sonrası kod farklı bir thread'de çalıştığında "TransactionScope must be disposed on the same thread" hatası alınır veya transaction sessizce iptal olur.36

C#

```
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    await connection.ExecuteAsync(sql1);
    await connection.ExecuteAsync(sql2);
    scope.Complete(); // Commit işlemi
}
```

---

## 8. Mimari Desenler ve Ekosistem Eklentileri

### 8.1 Repository Pattern Tartışması

Dapper ile Repository Pattern kullanımı tartışmalı bir konudur. Dapper zaten bir soyutlamadır. Ancak test edilebilirlik (mocking) ve kod tekrarını önlemek adına, özellikle "Generic Repository" yaklaşımı sıkça uygulanır. Ancak karmaşık sorgular için özel repository metodları yazmak, "Leaky Abstraction" (sızdıran soyutlama) problemini önlemek için daha sağlıklıdır.25

### 8.2 Dapper.Contrib ile CRUD Otomasyonu

Her tablo için tek tek `INSERT INTO...` yazmak zaman alıcıdır. `Dapper.Contrib` kütüphanesi, model üzerine eklenen `[Key]`, `` gibi nitelikleri (attributes) kullanarak temel CRUD işlemlerini otomatikleştirir.

- `connection.Insert(new Urun {... })`
    
- `connection.Get<Urun>(1)`
    
- `connection.Update(new Urun {... })`
    

**Önemli Not:** `Update` işlemi sırasında `` veya `[Computed]` olarak işaretlenen alanlar güncellenmez. Ayrıca, SQLite kullanılırken `Update` metodu SQLite'ın kendi metoduyla çakışabilir; bu durumda `SqlMapperExtensions.Update` şeklinde açık çağrı yapılmalıdır.7

### 8.3 Z.Dapper.Plus ve Toplu İşlemler

Ücretsiz eklentilerin yanı sıra, ticari bir kütüphane olan `Z.Dapper.Plus`, milyonlarca satırlık veriyi `BulkInsert`, `BulkUpdate` metodlarıyla çok yüksek performansla işleyebilir. Standart Dapper ile döngü içinde tek tek insert yapmak yerine bu tür kütüphaneler veya TVP kullanmak performans için kritiktir.39

---

## 9. Hata Toleransı ve Dayanıklılık (Resilience)

Mikroservisler, ağ hatalarının ve geçici veritabanı kesintilerinin (Transient Faults) kaçınılmaz olduğu ortamlardır. Dapper, yerleşik bir "Retry" (yeniden deneme) mekanizmasına sahip değildir. Bu eksiklik, **Polly** kütüphanesi ile giderilir.

Özellikle bulut ortamlarında (Azure SQL, AWS RDS), veritabanı bağlantısı milisaniyelik kopmalar yaşayabilir. `SqlException` hatalarını yakalayan ve belirli hata kodlarına (timeout, deadlock) göre işlemi tekrar eden bir politika oluşturulmalıdır.40

C#

```
// Geçici hataları (Transient) algılayan ve üstel bekleme (Exponential Backoff) yapan politika
var retryPolicy = Policy
   .Handle<SqlException>(ex => IsTransient(ex)) // IsTransient metodu hata kodlarını kontrol eder
   .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));

await retryPolicy.ExecuteAsync(() => connection.QueryAsync(...));
```

Bu desen, uygulamanın kendi kendini iyileştirmesini (self-healing) sağlar ve kullanıcıya hata göstermek yerine arka planda sorunu çözer.42

---

## 10. Kalite Güvencesi: Test ve İzleme

### 10.1 Unit Testing Zorlukları ve Çözümler

Dapper metodları (`Query`, `Execute`), `IDbConnection` üzerindeki statik genişletme metodlarıdır (Extension Methods). C# dilinde statik metodları `Moq` gibi kütüphanelerle "Mock"lamak (taklit etmek) zordur. Bu durum, Unit Test yazmayı zorlaştırır.

**Çözüm Stratejileri:**

1. **Wrapper Pattern:** Dapper çağrılarını yapan bir `IDapperContext` veya `IDbExecutor` arayüzü oluşturulur ve testlerde bu arayüz mocklanır. Bu yöntem iş mantığını test eder ancak SQL'in doğruluğunu test etmez.44
    
2. **Integration Testing (Önerilen):** Docker üzerinde çalışan gerçek bir veritabanı (örn: **Testcontainers**) kullanılarak yapılan testlerdir. Dapper'ın yazdığınız SQL'i gerçekten çalıştırıp çalıştırmadığını doğrulamanın en güvenilir yolu budur.46
    
3. **In-Memory SQLite:** Hızlı testler için SQLite in-memory modu kullanılabilir ancak SQL Server ile sözdizimi farklılıkları (tarih formatları vb.) yanlış pozitif sonuçlara yol açabilir.7
    

### 10.2 Logging (Günlükleme)

EF Core otomatik olarak oluşturduğu SQL'i loglar. Dapper ise "ne verirsen onu çalıştırır", loglama yapmaz. SQL sorgularını ve çalışma sürelerini loglamak için `IDbConnection` nesnesini sarmalayan (Decorator Pattern) özel bir yapı kurulabilir. `Dapper.Logging` gibi kütüphaneler veya MiniProfiler, çalıştırılan SQL'i ve parametreleri `ILogger` üzerinden loglayarak performans darboğazlarını tespit etmenize yardımcı olur.6

---

## 11. Sonuç

Dapper,.NET Core mikroservis ekosisteminde performans, kontrol ve şeffaflığın simgesidir. Tam kapsamlı ORM'lerin sunduğu konforun yerine, geliştiriciye "veriye nasıl erişileceği" konusunda tam yetki verir. Bu rapor boyunca incelenen bağlantı yönetimi, asenkron programlama, parametre güvenliği ve hata toleransı teknikleri, sadece Dapper'ı kullanmayı değil, onu "üretim ortamına hazır" (production-ready) hale getirmeyi öğretmektedir.

Bir mikroservis geliştiricisi için Dapper, sadece bir kütüphane değil, SQL diline ve veritabanı mimarisine hakimiyetin bir göstergesidir. Doğru mimari desenlerle (Scoped Connection, Polly Retries, Integration Tests) desteklendiğinde, Dapper ile inşa edilen sistemler, en yüksek trafik yükleri altında bile kararlı ve hızlı çalışmaya devam edecektir.

