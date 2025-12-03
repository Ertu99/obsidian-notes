## 1️⃣ `OUTPUT` Nedir?

> SQL’de OUTPUT, bir komutun (INSERT, UPDATE, DELETE veya MERGE)
> 
> **etkilenen satırlarını** ya da **değişen değerlerini** anlık olarak döndürmek için kullanılır.

Yani:

- Ne eklendi?
    
- Ne silindi?
    
- Eski değer neydi, yeni değer ne oldu?
    
    → hepsini `OUTPUT` ile görebilirsin.
    

---

## 💻 2️⃣ En Basit Örnek – INSERT’te OUTPUT

```sql
CREATE TABLE Employees
(
    Id INT IDENTITY PRIMARY KEY,
    FirsName NVARCHAR(50),
    Salary DECIMAL(10,2)
);

```

Şimdi yeni çalışan ekleyelim ve eklenen kaydı anında görelim 👇

```sql
INSERT INTO Employees (FirsName, Salary)
OUTPUT inserted.Id, inserted.FirsName, inserted.Salary
VALUES ('Ali', 8000), ('Ayşe', 9500);

```

### 🔍 Çıktı:

|Id|FirsName|Salary|
|---|---|---|
|1|Ali|8000.00|
|2|Ayşe|9500.00|

🧠 `inserted` tablosu, **eklenen yeni kayıtları** geçici olarak tutar.

---

## ⚙️ 3️⃣ UPDATE’te OUTPUT

`UPDATE` yaparken **hem eski hem yeni değerleri** aynı anda görebilirsin 👇

```sql
UPDATE Employees
SET Salary = Salary * 1.10
OUTPUT deleted.FirsName, deleted.Salary AS OldSalary,
       inserted.Salary AS NewSalary;

```

### 🔍 Çıktı:

|FirsName|OldSalary|NewSalary|
|---|---|---|
|Ali|8000.00|8800.00|
|Ayşe|9500.00|10450.00|

🧠

- `deleted` → eski değerleri tutar
- `inserted` → yeni değerleri tutar

Yani `OUTPUT` sayesinde tabloyu elle sorgulamadan değişiklikleri anında görebilirsin ✅

---

## 🧱 4️⃣ DELETE’te OUTPUT

Silinen kayıtları görmek için `deleted` kullanılır 👇

```sql
DELETE FROM Employees
OUTPUT deleted.Id, deleted.FirsName, deleted.Salary
WHERE FirsName = 'Ali';

```

### 🔍 Çıktı:

|Id|FirsName|Salary|
|---|---|---|
|1|Ali|8800.00|

🧠 Burada `deleted` sadece silinen satırları içerir.

Yani, bir tür “log” gibi kullanılabilir.

---

## 💾 5️⃣ OUTPUT’u başka tabloya yazmak (en faydalı kullanım)

OUTPUT sadece ekranda göstermek için değil,

aynı zamanda **log tablosuna kayıt** atmak için de kullanılabilir 👇

### Örnek:

```sql
CREATE TABLE EmployeeLog
(
    EmployeeId INT,
    OldSalary DECIMAL(10,2),
    NewSalary DECIMAL(10,2),
    ChangeDate DATETIME DEFAULT GETDATE()
);

UPDATE Employees
SET Salary = Salary * 1.10
OUTPUT deleted.Id, deleted.Salary, inserted.Salary
INTO EmployeeLog(EmployeeId, OldSalary, NewSalary);

```

🧠 Burada:

- `OUTPUT ... INTO EmployeeLog` → değişen kayıtları log tablosuna ekler.
- Yani “kim ne kadar zam aldı” gibi bilgileri arşivlersin.

---

## ⚙️ 6️⃣ OUTPUT Kullanımı Stored Procedure İçinde

Stored Procedure içinde OUTPUT iki şekilde kullanılır:

1. **Tablo değişikliklerini döndürmek için (inserted/deleted)**
2. **Parametre olarak dışarı değer döndürmek için**

---

### 🧩 a) Tablo değişikliklerini döndürme

```sql
CREATE PROCEDURE AddEmployee
AS
BEGIN
    INSERT INTO Employees (FirsName, Salary)
    OUTPUT inserted.Id, inserted.FirsName
    VALUES ('Elif', 12000);
END;

```

Çalıştırınca:

```sql
EXEC AddEmployee;

```

Eklenecek satırın bilgilerini döndürür 👇

|Id|FirsName|
|---|---|
|3|Elif|

---

### 🧩 b) OUTPUT parametresiyle değer döndürme

> Bu, SP’lerde RETURN değeri dışında “çıktı parametresi” kullanmak anlamına gelir.

```sql
CREATE PROCEDURE GetEmployeeCount
    @TotalCount INT OUTPUT
AS
BEGIN
    SELECT @TotalCount = COUNT(*) FROM Employees;
END;

```

Kullanımı:

```sql
DECLARE @Count INT;
EXEC GetEmployeeCount @TotalCount = @Count OUTPUT;
PRINT 'Toplam çalışan sayısı: ' + CAST(@Count AS NVARCHAR(10));

```

---

## 🧠 7️⃣ OUTPUT Kullanımında Bilmen Gereken 3 Mini Tablo

|Sanal tablo|Ne içerir|Nerede kullanılır|
|---|---|---|
|**inserted**|Yeni değerler|INSERT, UPDATE|
|**deleted**|Eski değerler|UPDATE, DELETE|
|**inserted + deleted**|Her iki hali|UPDATE|

---

## ⚖️ 8️⃣ OUTPUT’un Kullanım Alanları

|Alan|Kullanım|
|---|---|
|Loglama|Kim ne zaman değişti?|
|Otomasyon|Değişiklik sonrası trigger gibi işlem|
|Performans|Tek sorguda eski–yeni karşılaştırma|
|Debug / Audit|Denetim tablosu (EmployeeLog gibi)|

---

## ✅ 9️⃣ Kısa Özet (not defterine yazmalık)

> 🔹 OUTPUT, bir SQL komutunun etkilediği satırları veya değerleri gösterir.
> 
> 🔹 `inserted` → yeni değerleri, `deleted` → eski değerleri tutar.
> 
> 🔹 INSERT, UPDATE, DELETE ve MERGE komutlarıyla kullanılabilir.
> 
> 🔹 `OUTPUT ... INTO` ile sonuçları başka tabloya yazabilirsin.
> 
> 🔹 Stored Procedure içinde hem tablo sonucu hem parametre sonucu olarak kullanılabilir.