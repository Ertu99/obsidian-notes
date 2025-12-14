## 1️⃣ Stored Procedure Nedir?

**Stored Procedure**,

> Veritabanında önceden yazılmış ve saklanmış SQL komutları bütünüdür.

Yani, sık sık çalıştırdığın bir SQL sorgusunu kod içinde tekrar tekrar yazmak yerine,

**veritabanında bir kere tanımlarsın** ve sadece ismini çağırırsın.

---

### 💡 Basit tanım:

> Stored procedure = veritabanı içinde çalışan hazır fonksiyon gibidir.

---

## 🧩 2️⃣ Ne işe yarar?

- Sorguları **merkezi** hale getirir (her yerde aynı prosedürü çağırırsın).
- **Performans kazandırır** (ilk çalışmada derlenir ve plan bellekte tutulur).
- **Güvenliği artırır** (veritabanına doğrudan sorgu gönderilmez).
- Kod tekrarını azaltır.
- **Parametre** alabilir (örneğin: şehir = “Istanbul”).

---

## 🧩 3️⃣ Örnek – Basit Stored Procedure

### 🔹 SQL Server tarafında:

```sql
CREATE PROCEDURE GetAllHotels
AS
BEGIN
    SELECT Id, Name, City, Star
    FROM Hotels
END

```

### 🔹 Çağırma:

```sql
EXEC GetAllHotels

```

---

## 🧩 4️⃣ Parametreli Stored Procedure

### 🔹 Oluşturma:

```sql
CREATE PROCEDURE GetHotelsByCity
    @City NVARCHAR(50)
AS
BEGIN
    SELECT Id, Name, City, Star
    FROM Hotels
    WHERE City = @City
END

```

### 🔹 Çağırma:

```sql
EXEC GetHotelsByCity @City = 'Istanbul'

```

## ⚙️ 6️⃣ Stored Procedure’lerin Özellikleri

|Özellik|Açıklama|
|---|---|
|Çalışma yeri|Veritabanı içinde|
|Parametre alabilir mi|✅ Evet|
|Değer döndürebilir mi|✅ Evet (`OUTPUT` parametresi veya `RETURN`)|
|Performans|Yüksek (önceden derlenir)|
|Güvenlik|Daha yüksek (SQL injection riski azalır)|
|Güncellenebilirlik|Tek noktadan yönetilir|
|Dil|T-SQL (SQL Server), PL/pgSQL (PostgreSQL), PL/SQL (Oracle) vb.|

---

## 💬 7️⃣ Kısa Mülakat Cevabı:

> “Stored Procedure, veritabanında saklanan ve gerektiğinde çağrılan SQL komutları bütünüdür.
> 
> Parametre alabilir, performansı artırır ve güvenliği sağlar.
> 
> Kod tekrarını önlemek için backend yerine doğrudan veritabanı tarafında çalışır.”

## 🧩 1️⃣ Temel Tanımlar

### 🔹 **Stored Procedure (SP)**

> Veritabanında saklanan ve bir işi (sorgu, ekleme, silme, güncelleme) yapan komutlar bütünüdür.
> 
> Sonuç döndürebilir ama **asıl amacı işlem yapmak**tır.

### 🔹 **Function**

> Bir veya daha fazla parametre alıp, tek bir değer ya da tablo döndüren SQL bloğudur.
> 
> Asıl amacı **hesaplama veya veri döndürmektir.**

---

## ⚖️ 2️⃣ En Temel Fark (Kısa Özet)

|Özellik|**Stored Procedure**|**Function**|
|---|---|---|
|Amacı|İşlem yapmak (INSERT, UPDATE, DELETE, SELECT)|Hesaplama veya değer döndürmek|
|Dönüş tipi|Zorunlu değil (`RETURN` opsiyonel)|Zorunlu (tek değer veya tablo döndürür)|
|`SELECT` içinde kullanılabilir mi|❌ Hayır|✅ Evet|
|`INSERT`, `UPDATE`, `DELETE` yapabilir mi|✅ Evet|❌ Hayır|
|`OUTPUT` parametresi alabilir mi|✅ Evet|❌ Hayır|
|Kullanım yeri|İş mantığı (business logic)|Hesaplama / sorgu içinde|
|Transaction yönetimi|✅ Evet (BEGIN TRAN, COMMIT, ROLLBACK)|❌ Hayır|
|Performans|Yüksek (önceden derlenir)|Hafif (daha küçük işlemler için)|

---

## 🧩 3️⃣ Örneklerle Görelim

### 🔹 Stored Procedure

```sql
CREATE PROCEDURE AddHotel
    @Name NVARCHAR(100),
    @City NVARCHAR(50)
AS
BEGIN
    INSERT INTO Hotels (Name, City)
    VALUES (@Name, @City)
END

```

🧠 Açıklama:

- Tabloya kayıt ekliyor.
- Dönüş değeri yok (bir işlem yapıyor).

Kullanımı:

```sql
EXEC AddHotel @Name = 'Hilton', @City = 'Istanbul'

```

---

### 🔹 Function (Scalar – tek değer döndüren)

```sql
CREATE FUNCTION GetHotelCountByCity(@City NVARCHAR(50))
RETURNS INT
AS
BEGIN
    DECLARE @Count INT
    SELECT @Count = COUNT(*) FROM Hotels WHERE City = @City
    RETURN @Count
END

```

Kullanımı:

```sql
SELECT dbo.GetHotelCountByCity('Istanbul') AS TotalHotels

```

🧠 Açıklama:

- Sadece bir sayı (INT) döndürüyor.
- `SELECT` içinde çağrılabiliyor.
- Ama tabloya kayıt ekleyemez.

---

### 🔹 Table-Valued Function (tablo döndüren)

```sql
CREATE FUNCTION GetHotelsByCity(@City NVARCHAR(50))
RETURNS TABLE
AS
RETURN
(
    SELECT * FROM Hotels WHERE City = @City
)

```

Kullanımı:

```sql
SELECT * FROM dbo.GetHotelsByCity('Istanbul');

```

---

## 🧠 4️⃣ Hangisini Ne Zaman Kullanırız?

|Durum|Hangisi Kullanılır|Neden|
|---|---|---|
|Veri ekleme / güncelleme|**Stored Procedure**|Transaction yönetimi yapılabilir|
|Hesaplama, sorguda kullanılacak küçük işlem|**Function**|SELECT içinde çağrılabilir|
|Raporlama / veri analizi|Function|Değer döndürür|
|API veya backend’ten veritabanına komut gönderme|Stored Procedure|Gelişmiş işlem yapabilir|

---

## 💬 5️⃣ Kısa Mülakat Cevabı:

> “Stored Procedure, işlem yapmak için (insert, update, delete) kullanılır ve değer döndürmek zorunda değildir.
> 
> Function ise hesaplama yapar ve **her zaman bir değer döndürür.**
> 
> Ayrıca Function, SELECT içinde kullanılabilir ama Procedure kullanılamaz.
> 
> Procedure transaction yönetebilir, function edemez.”