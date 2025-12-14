


## 1. `IQueryable` nedir?

`IQueryable<T>` aslında **hazır ama çalışmamış bir sorgudur**.

Yani EF Core’da yazdığın sorgular **hemen çalışmaz**, “sorgu planı” olarak tutulur.

🧠 Basit tanım:

> IQueryable, veritabanında çalışacak sorgunun temsilidir.
> 
> Sen ne zaman “veriyi gerçekten istiyorsan” (`ToList()`, `FirstOrDefault()`, `Count()`)
> 
> o zaman SQL’e çevrilip çalıştırılır.

---

## 🧩 2. `IQueryable`’ın rolü EF Core’da

EF Core, LINQ sorgularını alır → SQL’e çevirir → veritabanına gönderir.

Ama bunu _hemen yapmaz._

Örneğin 👇

```csharp
var query = _db.Hotels.Where(h => h.City == "Istanbul");

```

Burada `query` değişkeni aslında **IQueryable<Hotel>** türündedir.

Henüz hiçbir SQL sorgusu gitmez ❌

Ancak sen şu satırı yazdığında:

```csharp
var hotels = await query.ToListAsync();

```

EF Core sorguyu SQL’e çevirir ve çalıştırır ✅

📜 Oluşan SQL:

```sql
SELECT * FROM Hotels WHERE City = 'Istanbul';

```

---

## 🧠 3. Neden “hazır ama çalışmamış sorgu” deniyor?

Çünkü `IQueryable`:

- Sorguyu **oluşturur ama çalıştırmaz.**
    
- Filtreleri, sıralamaları, `Include`’ları bir araya getirip bekletir.
    
- Ne zaman “sonuç” istersin (`ToList`, `Count`, `Any`, `FirstOrDefault`),
    
    o zaman **tek SQL sorgusuna çevirip** çalıştırır.
    

---

## 🔹 4. Örnek fark

```csharp
// IQueryable
var query = _db.Hotels
    .Where(h => h.Star >= 4)
    .Include(h => h.Rooms);

```

➡️ Bu satırda **henüz sorgu çalışmadı.**

`query` hâlâ “planlanmış” sorgudur.

Sonra:

```csharp
var hotels = await query.ToListAsync();

```

Bu satırda sorgu **veritabanında çalışır**.

---

## 🧩 5. `IQueryable` vs `IEnumerable` farkı

|Özellik|`IQueryable`|`IEnumerable`|
|---|---|---|
|Nerede çalışır?|**Veritabanında (SQL)**|**Bellekte (C# tarafında)**|
|Ne zaman çalışır?|Sorgu “execute” edilince|Veriler çekildikten sonra|
|Performans|Yüksek (SQL tarafında filtrelenir)|Düşük (önce tüm veriyi çeker)|
|Örnek|`.Where()` (henüz çalışmaz)|`.Where()` (veriler bellekte filtrelenir)|

💡 `IQueryable`, LINQ sorgusunu SQL olarak çalıştırır.

💡 `IEnumerable`, veriyi çektikten sonra bellekte işler.

---

## 🔹 6. Repository’de neden kullanılır?

Birçok repository metodu `IQueryable` döner,

çünkü üst katman (Application veya Service) ek filtreler uygulamak isteyebilir.

Örneğin 👇

```csharp
// Repository
public IQueryable<Hotel> GetAll()
{
    return _db.Hotels.AsQueryable();
}

```

Şimdi Application katmanında:

```csharp
var hotels = await _hotelRepository
    .GetAll()
    .Where(h => h.City == "Istanbul")
    .OrderBy(h => h.Star)
    .ToListAsync();

```

Burada hem repository’nin döndürdüğü `IQueryable`

hem Application katmanındaki `Where` zincirlenir →

EF Core hepsini **tek SQL sorgusu** olarak üretir 👇

```sql
SELECT * FROM Hotels
WHERE City = 'Istanbul'
ORDER BY Star;

```

Yani `IQueryable` sayesinde sorgular **birleştirilebilir ve optimize edilir**.

---

## 🧠 7. Özet

|Özellik|Açıklama|
|---|---|
|**IQueryable**|Henüz çalışmamış sorgudur (sorgu ifadesini temsil eder)|
|**Ne işe yarar?**|SQL sorgularını dinamik olarak oluşturup zincirlemeyi sağlar|
|**Ne zaman çalışır?**|`ToList()`, `FirstOrDefault()`, `Count()`, vs. çağrıldığında|
|**Avantajı**|Filtreler tek sorguda birleşir → performans artar|
|**Repository’deki rolü**|Üst katmanın sorguya ekstra filtre eklemesine izin verir|

---

💬 Kısaca:

> IQueryable = “henüz çalışmamış, ama SQL’e çevrilmeye hazır sorgu”.
> 
> Bu sayede sorguları zincirleyebilir, filtreleri dinamik oluşturabilir
> 
> ve EF Core bunları **tek SQL sorgusunda** çalıştırabilir. ✅

## 🏨 Örnek Senaryo

Elimizde `Hotels` tablosu var:

|Id|Name|City|Star|
|---|---|---|---|
|1|Hilton|İstanbul|5|
|2|Radisson|Ankara|4|
|3|Ibis|İstanbul|3|

---

## ⚙️ Kod 1 — IQueryable (Veritabanında filtreleme)

```csharp
var query = _db.Hotels
    .Where(h => h.Star >= 4);   // IQueryable oluşturulur

var result = await query.ToListAsync();   // 🔹 ŞİMDİ sorgu çalışır!

```

🧠 EF Core burada sorguyu “hazırladı” ama `.ToListAsync()` çağrısına kadar bekledi.

Yani **filtre veritabanında** uygulanır.

📜 Oluşan SQL:

```sql
SELECT [h].[Id], [h].[Name], [h].[City], [h].[Star]
FROM [Hotels] AS [h]
WHERE [h].[Star] >= 4;

```

💡 Veritabanı sadece 5 ve 4 yıldızlı otelleri getirir → **az veri**, **yüksek performans**.

---

## ⚙️ Kod 2 — IEnumerable (Bellekte filtreleme)

```csharp
var list = await _db.Hotels.ToListAsync();   // 🔸 Sorgu HEMEN çalıştı
var result = list.Where(h => h.Star >= 4);   // Filtreleme C# tarafında yapılıyor

```

📜 Oluşan SQL:

```sql
SELECT [h].[Id], [h].[Name], [h].[City], [h].[Star]
FROM [Hotels] AS [h];

```

💡 EF Core burada **bütün kayıtları** çekti (filtre yok!).

Sonra `Where` işlemi **C# belleğinde** uygulandı.

Yani 100.000 kayıt varsa, hepsi RAM’e gelir, sonra filtrelenir → **performans düşer**.

---

## 📊 Farkın Özeti

|Özellik|IQueryable|IEnumerable|
|---|---|---|
|Sorgu ne zaman çalışır?|`ToListAsync()` çağrıldığında|Veritabanından tüm veri alındığında|
|Filtre nerede uygulanır?|**SQL tarafında (server)**|**C# tarafında (client)**|
|Performans|Yüksek|Düşük (büyük tablolarda yavaş)|
|SQL çıktısı|Filtre içerir|Filtre içermez|
|Ne zaman kullanılır|EF Core sorgularında|Belleğe alınmış listelerle çalışırken|

---

## 🧠 Kısaca hatırla:

> 🔹 IQueryable = “hazır, ama henüz çalışmamış sorgu” → SQL’e dönüşür.
> 
> 🔹 `IEnumerable` = “artık veriler bellekte” → **LINQ C#’ta çalışır.**