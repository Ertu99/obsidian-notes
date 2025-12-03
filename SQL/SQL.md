## 🎯 1️⃣ SQL’in En Temel Komutları (CRUD)

Bunları zaten öğrendin ama tekrar kısa özetle:

| Komut    | Açıklama           | Örnek                                       |
| -------- | ------------------ | ------------------------------------------- |
| `SELECT` | Veri çekmek        | `SELECT * FROM Employees;`                  |
| `INSERT` | Yeni veri eklemek  | `INSERT INTO Employees (...) VALUES (...);` |
| `UPDATE` | Veriyi güncellemek | `UPDATE Employees SET Age=30 WHERE Id=1;`   |
| `DELETE` | Veriyi silmek      | `DELETE FROM Employees WHERE Id=1;`         |

✅ Bu dört komut SQL’in temeli.

---

## 🧱 2️⃣ Filtreleme ve Koşullar

Bu kısımda WHERE ve operatörleri iyice öğrenmen gerek:

|Yapı|Açıklama|Örnek|
|---|---|---|
|`WHERE`|Filtreleme yapar|`WHERE City = 'Istanbul'`|
|`AND` / `OR`|Çoklu koşul|`WHERE Age > 25 AND City='Istanbul'`|
|`IN`|Birden fazla değeri kontrol eder|`WHERE City IN ('Istanbul', 'Ankara')`|
|`BETWEEN`|Aralık sorgusu|`WHERE Age BETWEEN 25 AND 35`|
|`LIKE`|Metin arama|`WHERE Name LIKE 'A%'`|
|`IS NULL` / `IS NOT NULL`|Boş değer kontrolü|`WHERE DeptId IS NULL`|

## ⚙️ 3️⃣ Sıralama, Limit ve Tekrarsız Listeleme

| Komut           | Açıklama                       | Örnek                                  |
| --------------- | ------------------------------ | -------------------------------------- |
| `ORDER BY`      | Sıralama                       | `ORDER BY Age DESC`                    |
| `TOP` / `LIMIT` | Belirli sayıda veri getirir    | `SELECT TOP 3 * FROM Employees`        |
| `DISTINCT`      | Tekrarlayan kayıtları kaldırır | `SELECT DISTINCT City FROM Employees;` |

---

## 📊 4️⃣ Toplama ve Gruplama (Raporlama Mantığı)

|Komut|Açıklama|Örnek|
|---|---|---|
|`COUNT()`|Kayıt sayısı|`SELECT COUNT(*) FROM Employees;`|
|`SUM()`|Toplam|`SELECT SUM(Salary) FROM Employees;`|
|`AVG()`|Ortalama|`SELECT AVG(Age) FROM Employees;`|
|`MIN()` / `MAX()`|En küçük / en büyük|`SELECT MAX(Age) FROM Employees;`|
|`GROUP BY`|Gruplama|`GROUP BY DeptId`|
|`HAVING`|Gruplara filtre uygular|`HAVING COUNT(*) > 2`|

✅ Bu kısım “raporlama sorguları” içindir.

---

## 🔗 5️⃣ JOIN’ler (Tabloları birleştirme)

Bunları çok iyi öğrendin zaten ama özetleyelim:

|JOIN türü|Açıklama|
|---|---|
|`INNER JOIN`|Eşleşen kayıtlar|
|`LEFT JOIN`|Sol tablo + eşleşen sağ taraf|
|`RIGHT JOIN`|Sağ tablo + eşleşen sol taraf|
|`FULL OUTER JOIN`|Her iki tablonun tümü|
|`CROSS JOIN`|Her satırı diğeriyle çarpar|

## 🧩 6️⃣ Alt Sorgular (Subquery)

- Sorgu içinde başka sorgu kullanma:

```sql
SELECT Name
FROM Employees
WHERE DeptId = (SELECT Id FROM Departments WHERE DepartmentName = 'IT');

```

## 💾 7️⃣ Veri Tasarımı (Constraints)

| Tür           | Açıklama                                            |
| ------------- | --------------------------------------------------- |
| `PRIMARY KEY` | Her satırı benzersiz tanımlar                       |
| `FOREIGN KEY` | Başka tabloyla ilişki kurar                         |
| `UNIQUE`      | Aynı değer tekrar edemez                            |
| `DEFAULT`     | Varsayılan değer verir                              |
| `CHECK`       | Değerin belirli bir aralıkta olmasını zorunlu kılar |

✅ Bu yapılar veritabanı bütünlüğünü korur.

SELECT * FROM Products ORDER BY Id OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY;

Yani:

- OFFSET → atlanacak kayıt sayısı (`(page - 1) * pageSize`)
- FETCH NEXT → alınacak kayıt sayısı (`pageSize`)