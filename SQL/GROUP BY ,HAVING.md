## 🎯 1️⃣ GROUP BY Nedir?

> GROUP BY, aynı değerlere sahip satırları gruplamak için kullanılır.
> 
> Genellikle **SUM**, **COUNT**, **AVG**, **MIN**, **MAX** gibi **aggregate (toplu)** fonksiyonlarla birlikte kullanılır.

Yani:

> “Departman bazında toplam maaşları getir.”
> 
> “Şehre göre çalışan sayısını bul.”
> 
> gibi sorgularda kullanılır.

---

## 💻 2️⃣ Örnek tablo

|Id|FirsName|DeptId|Salary|
|---|---|---|---|
|1|Ali|10|8000|
|2|Ayşe|10|9000|
|3|Mehmet|20|9500|
|4|Elif|20|11000|
|5|Hasan|30|7000|

---

## 💡 3️⃣ GROUP BY Temel Kullanım

### 🎯 “Departmana göre ortalama maaşları getir.”

```sql
SELECT DeptId, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY DeptId;

```

### 🔍 Çıktı:

|DeptId|AvgSalary|
|---|---|
|10|8500|
|20|10250|
|30|7000|

🧠 `GROUP BY` olmasaydı, `AVG()` fonksiyonu tüm tabloyu tek bir grup olarak hesaplardı.

Ama `GROUP BY DeptId` dediğimizde, **her departman ayrı hesaplanır.**

---

## ⚙️ 4️⃣ Birden Fazla Kolonla GROUP BY

> Birden fazla kolona göre gruplama da mümkündür.

Örneğin:

```sql
SELECT DeptId, City, COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY DeptId, City;

```

Bu durumda hem `DeptId` hem `City` kolonlarına göre gruplama yapılır.

---

## 🧩 5️⃣ HAVING Nedir?

> HAVING, GROUP BY sonrasında gelen gruplar üzerinde filtreleme yapmak için kullanılır.
> 
> Yani `WHERE` gibi davranır ama **aggregate fonksiyonlarla birlikte** çalışır.

### 🎯 “Ortalama maaşı 9000’den yüksek olan departmanları getir.”

```sql
SELECT DeptId, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY DeptId
HAVING AVG(Salary) > 9000;

```

### 🔍 Çıktı:

|DeptId|AvgSalary|
|---|---|
|20|10250|

🧠 `WHERE` ile `AVG(Salary)` kullanamazsın çünkü `AVG` hesaplanmadan önce satırlar gruplandırılmamıştır.

Bu yüzden aggregate filtreleri **HAVING** ile yaparız.

---

## ⚖️ 6️⃣ WHERE vs HAVING Farkı

|Özellik|**WHERE**|**HAVING**|
|---|---|---|
|Çalışma zamanı|Gruplamadan **önce**|Gruplamadan **sonra**|
|Kullanıldığı yer|Satırları filtreler|Grupları filtreler|
|Aggregate fonksiyonlar|Kullanılamaz|Kullanılabilir|
|Örnek|`WHERE Salary > 9000`|`HAVING AVG(Salary) > 9000`|

---

## 💻 7️⃣ WHERE + GROUP BY + HAVING birlikte örnek

### 🎯 “Departman bazında çalışan sayısını getir,

ama sadece maaşı 8000’den büyük olan çalışanları dikkate al,

ve sayısı 1’den fazla olan departmanları göster.”

```sql
SELECT DeptId, COUNT(*) AS EmpCount
FROM Employees
WHERE Salary > 8000         -- önce satırları filtreler
GROUP BY DeptId             -- sonra gruplar
HAVING COUNT(*) > 1;        -- sonra grupları filtreler

```

### 🔍 Çıktı:

|DeptId|EmpCount|
|---|---|
|10|2|
|20|2|

---

## ⚙️ 8️⃣ HAVING ile SUM/COUNT/AVG örnekleri

|Sorgu|Anlamı|
|---|---|
|`HAVING COUNT(*) > 5`|5’ten fazla kaydı olan grupları getir|
|`HAVING SUM(Salary) > 50000`|Toplam maaşı 50.000’den fazla olan gruplar|
|`HAVING MIN(Salary) < 3000`|En düşük maaşı 3000’den küçük olan gruplar|

---

## 🔍 9️⃣ ORDER BY ile birlikte kullanımı

> GROUP BY sonrası sonuçları sıralamak da mümkündür:

```sql
SELECT DeptId, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY DeptId
ORDER BY TotalSalary DESC;

```

---

## 🧠 10️⃣ Önemli Bilgi: GROUP BY’da sadece “grup sütunları” seçilebilir

Aşağıdaki yanlış olur 👇

```sql
SELECT FirsName, SUM(Salary) FROM Employees GROUP BY DeptId;

```

Çünkü `FirsName` gruplama içinde değil.

Doğru hali 👇

```sql
SELECT DeptId, SUM(Salary) FROM Employees GROUP BY DeptId;

```

---

## ✅ 11️⃣ Kısa Özet (not defterine yazmalık)

> 🔹 GROUP BY → verileri belirli kolonlara göre gruplar.
> 
> 🔹 `HAVING` → gruplar üzerinde filtreleme yapar.
> 
> 🔹 `WHERE` → satır bazında, `HAVING` → grup bazında filtreleme yapar.
> 
> 🔹 `AVG`, `SUM`, `COUNT`, `MIN`, `MAX` gibi fonksiyonlarla birlikte kullanılır.
> 
> 🔹 `HAVING` her zaman `GROUP BY`’dan sonra gelir.