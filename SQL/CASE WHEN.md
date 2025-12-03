## 🧩 1️⃣ CASE WHEN Nedir?

> CASE WHEN, SQL’de koşula göre farklı değer döndürmek için kullanılır.
> 
> Yani **if / else if / else** yapısının SQL karşılığıdır.

---

## 💡 Temel yapı (genel syntax)

```sql
SELECT
    Column1,
    CASE
        WHEN koşul1 THEN sonuç1
        WHEN koşul2 THEN sonuç2
        ELSE varsayılan_sonuç
    END AS YeniSütun
FROM tablo_adi;

```

---

## 🧱 2️⃣ Basit örnek

### 🎯 Amaç:

Yaş bilgisine göre çalışanları **“Genç”**, **“Yetişkin”**, **“Yaşlı”** olarak sınıflandıralım.

```sql
SELECT
    FirsName,
    Age,
    CASE
        WHEN Age < 30 THEN 'Genç'
        WHEN Age BETWEEN 30 AND 40 THEN 'Yetişkin'
        ELSE 'Yaşlı'
    END AS YasGrubu
FROM Employees;

```

### 🔍 Çıktı:

|FirsName|Age|YasGrubu|
|---|---|---|
|Ali|28|Genç|
|Ayşe|35|Yetişkin|
|Mehmet|42|Yaşlı|
|Elif|25|Genç|

---

## ⚙️ 3️⃣ Başka örnek – NULL değerleri kontrol etmek

### 🎯 Amaç:

Departmanı olmayan çalışanlara “Atanmamış” yazalım.

```sql
SELECT
    FirsName,
    CASE
        WHEN DeptId IS NULL THEN 'Atanmamış'
        ELSE 'Atanmış'
    END AS DepartmanDurumu
FROM Employees;

```

### 🔍 Çıktı:

|FirsName|DepartmanDurumu|
|---|---|
|Ali|Atanmış|
|Ayşe|Atanmış|
|Mehmet|Atanmış|
|Elif|Atanmamış|

---

## 💬 4️⃣ Gerçek mülakat tarzı örnek

> “Çalışanların yaşına göre maaş zammı yüzdesini gösterin.”

```sql
SELECT
    FirsName,
    Age,
    CASE
        WHEN Age < 30 THEN 0.05   -- %5 zam
        WHEN Age BETWEEN 30 AND 40 THEN 0.10  -- %10 zam
        ELSE 0.15  -- %15 zam
    END AS ZamOrani
FROM Employees;

```

🧠 `CASE WHEN` çıktısı sadece yazı değil, **matematiksel işlem** de olabilir.

Hatta `UPDATE` içinde bile kullanılabilir 👇

---

## 🔄 5️⃣ CASE WHEN ile UPDATE

```sql
UPDATE Employees
SET Salary =
    CASE
        WHEN Age < 30 THEN Salary * 1.05
        WHEN Age BETWEEN 30 AND 40 THEN Salary * 1.10
        ELSE Salary * 1.15
    END;

```

> Bu sorgu yaş aralıklarına göre farklı oranlarda maaş artırır.

---

## 🧠 6️⃣ Kısa mülakat cevabı:

> “CASE WHEN, SQL’de koşula bağlı olarak farklı değerler döndürmek için kullanılır.
> 
> If–else mantığında çalışır, hem SELECT hem UPDATE sorgularında kullanılabilir.”