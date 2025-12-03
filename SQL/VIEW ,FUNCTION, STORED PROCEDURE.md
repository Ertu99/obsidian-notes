

## 🧩 1️⃣ VIEW (Sanal tablo)

### 🎯 Tanım:

> VIEW, bir SQL sorgusunun hazır kaydedilmiş halidir.
> 
> Sanki tabloymuş gibi davranır ama aslında **SELECT sorgusudur**.

---

### 💻 Örnek:

```sql
CREATE VIEW View_EmployeeDetails AS
SELECT e.FirsName, e.Age, d.DepartmentName
FROM Employees e
LEFT JOIN Departments d ON e.DeptId = d.Id;

```

> Artık bu sorguyu her seferinde yazmana gerek yok.
> 
> Sadece:

```sql
SELECT * FROM View_EmployeeDetails;

```

yazman yeterli ✅

---

### 🧠 Notlar:

- View = **sanal tablo** (veri tutmaz).
- Karmaşık sorguları kısaltmak ve performans artırmak için kullanılır.
- Genellikle **SELECT** işlemleri içindir (veri ekleme/güncelleme nadir yapılır).

---

## 🧩 2️⃣ FUNCTION (Fonksiyon)

### 🎯 Tanım:

> Function, bir işlem yapıp tek bir değer döndüren SQL yapısıdır.
> 
> C#’taki fonksiyon mantığı gibidir.

---

### 💻 Örnek – Scalar Function (tek değer döndürür)

```sql
CREATE FUNCTION GetFullName(@Id INT)
RETURNS NVARCHAR(100)
AS
BEGIN
    DECLARE @FullName NVARCHAR(100);
    SELECT @FullName = FirsName + ' ' + LastName FROM Employees WHERE Id = @Id;
    RETURN @FullName;
END;

```

> Kullanımı:

```sql
SELECT dbo.GetFullName(1);

```

🧠 Dönen değer: “Ali Yılmaz” gibi.

---

### 💡 Notlar:

- Fonksiyonlar **her zaman bir değer döndürür** (`RETURN`).
- `SELECT` içinde kullanılabilir.
- Sadece veri okumak içindir — tabloyu değiştiremez.
- Türleri:
    - **Scalar Function:** Tek değer döndürür (örneğin yaş, isim, ortalama).
    - **Table-Valued Function:** Tablo döndürür.

---

## 🧩 3️⃣ STORED PROCEDURE (Hazır işlem)

### 🎯 Tanım:

> Stored Procedure, içinde SQL komutları olan,
> 
> parametre alabilen ve **çok adımlı işlemler** yapabilen kod bloğudur.

---

### 💻 Örnek:

```sql
CREATE PROCEDURE GetEmployeesByCity
    @City NVARCHAR(50)
AS
BEGIN
    SELECT * FROM Employees WHERE City = @City;
END;

```

> Çalıştırmak için:

```sql
EXEC GetEmployeesByCity @City = 'Istanbul';

```

🧠 Parametre alabilir, güncelleme veya silme işlemi bile yapabilir.

---

## ⚖️ 4️⃣ Aralarındaki fark (mülakat tablosu)

|Özellik|VIEW|FUNCTION|STORED PROCEDURE|
|---|---|---|---|
|Ne işe yarar|Hazır SELECT sorgusu|İşlem yapar ve değer döndürür|SQL komutlarını çalıştırır|
|Veri döndürür mü|✅ Evet (tablo gibi)|✅ Evet (tek veya tablo)|✅ Evet (isteğe bağlı)|
|Veri günceller mi|⚠️ Genelde hayır|❌ Hayır|✅ Evet|
|Parametre alır mı|❌ Hayır|✅ Evet|✅ Evet|
|`SELECT` içinde kullanılabilir mi|✅ Evet|✅ Evet|❌ Hayır|
|Kullanım alanı|Görselleştirme, raporlama|Hesaplama, mantık|İşlemler, prosedürler|

---

## 🧠 5️⃣ Kısa mülakat cevabı

> “View sanal tablo gibidir, Function değer döndürür ama tabloyu değiştiremez,
> 
> Stored Procedure ise bir veya birden fazla SQL komutunu çalıştırır, parametre alabilir ve veri güncelleyebilir.”