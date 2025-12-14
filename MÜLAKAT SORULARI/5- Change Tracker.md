**Change Tracker**, EF Core’un “bellek içindeki verileri izleyen” sistemidir.

🧠 Tanım:

> EF Core, bellekte tuttuğu entity’lerin (nesnelerin) hangi durumda olduğunu,
> 
> **değişti mi**, **eklendi mi**, **silindi mi** gibi bilgileri takip eder.

Bu sayede sen `SaveChanges()` dediğinde EF Core:

- “Neyi INSERT edeceğim?”
    
- “Neyi UPDATE edeceğim?”
    
- “Neyi DELETE edeceğim?”
    
    gibi kararları **otomatik olarak verir.**
    

---

## 🧩 2. Her entity’nin bir “durumu (state)” vardır

Change Tracker, her entity’yi bir **EntityEntry** olarak tutar

ve bu entry’nin bir **State (durum)** değeri vardır.

|State|Anlamı|EF Core’un davranışı|
|---|---|---|
|**Added**|Yeni eklendi, DB’ye yazılmadı|INSERT üretir|
|**Modified**|Mevcut kayıt değiştirildi|UPDATE üretir|
|**Deleted**|Silindi olarak işaretlendi|DELETE üretir|
|**Unchanged**|Değişiklik yok|Hiçbir SQL üretmez|
|**Detached**|Takip edilmiyor|Change Tracker izlemez|

---

## 🧩 3. Senin örneğinde neler oluyor?

### Kod:

```csharp
_db.Hotels.Add(hotel);

```

Bu anda EF Core şunu yapar:

1. `hotel` nesnesini **Change Tracker**’a ekler.
2. Durumunu `EntityState.Added` olarak işaretler.
3. Henüz veritabanına **hiçbir SQL göndermez.**

Bunu görmek için şu kodu çalıştırabilirsin 👇

```csharp
Console.WriteLine(_db.Entry(hotel).State);
// Output: Added

```

---

## 🧩 4. “Henüz DB’ye gitmez” ne demek?

EF Core, işlemleri **hemen veritabanına göndermez.**

Sen `SaveChanges()` veya `SaveChangesAsync()` dediğinde:

- Change Tracker’daki bütün `Added`, `Modified`, `Deleted` durumundaki entity’leri toplar,
- uygun SQL’leri oluşturur (INSERT, UPDATE, DELETE),
- veritabanına **tek transaction içinde** gönderir.

Bu yaklaşım performansı artırır ve **birden fazla değişikliği tek seferde kaydetmeye** izin verir.

---

## 🧩 5. `SaveChanges()` çağrılınca ne olur?

Senin örneğinde:

```csharp
await _db.SaveChangesAsync();

```

1. EF Core Change Tracker’a bakar:
    
    → “Hotel entity’si Added durumda.”
    
2. INSERT sorgusu üretir:
    
    ```sql
    INSERT INTO Hotels (Name, City, Star)
    VALUES ('Hilton', 'Paris', 5);
    
    ```
    
3. Veritabanı yeni **Id** değerini döndürür (örneğin 42).
    
4. EF Core bu `Id` değerini **[hotel.Id](http://hotel.Id)** property’sine otomatik yazar.
    
5. Entity’nin durumu artık `Unchanged` olur (çünkü DB ile senkron hale geldi).
    

---

## 🧠 6. Change Tracker nasıl çalışıyor?

Arka planda EF Core, her entity’nin:

- Orijinal değerlerini,
    
- Güncel değerlerini,
    
- Navigation property’lerini
    
    bellekte tutar.
    

Örneğin:

```csharp
var hotel = await _db.Hotels.FirstAsync();
hotel.Name = "Hilton Istanbul";

```

Burada EF Core, değişikliği fark eder çünkü `hotel.Name` değişti.

Entity’nin state’i otomatik olarak `Modified` olur.

📊 Örnek:

```csharp
Console.WriteLine(_db.Entry(hotel).State);
// Output: Modified

```

Sonra `SaveChanges()` dediğinde EF Core sadece o property için SQL üretir:

```sql
UPDATE Hotels SET Name = 'Hilton Istanbul' WHERE Id = 1;

```

---

## 🧩 7. Ne zaman Change Tracker devreye girmez?

Eğer `.AsNoTracking()` kullanırsan:

```csharp
var hotels = _db.Hotels.AsNoTracking().ToList();

```

Bu durumda EF Core “ben bunları takip etmeyeceğim” der, Change Tracker’a eklemez.

Çünkü bu veri sadece **okunacak**, **değiştirilmeyecek**.

---

## 🧩 8. Özet

|Özellik|Açıklama|
|---|---|
|**Change Tracker**|EF Core’un entity’lerin durumlarını izleyen mekanizması|
|**Ne yapar?**|Entity Added, Modified, Deleted mi diye takip eder|
|**Ne zaman kaydeder?**|`SaveChanges()` çağrıldığında|
|**State**|Entity’nin veritabanına göre durumu|
|**AsNoTracking()**|Takip etmeyi kapatır (sadece okuma)|

---

💬 Kısaca:

> Change Tracker, EF Core’un bellek içinde “hangi entity’nin ne durumda olduğunu” izlediği mekanizmadır.
> 
> `_db.Hotels.Add(hotel)` dediğinde sadece “bu yeni eklenecek” diye işaretler,
> 
> **`SaveChangesAsync()`** dediğinde ise bu bilgiyi kullanıp uygun **INSERT** sorgusunu üretir.