## 🎯 1️⃣ Index Nedir?

> Index, veritabanında bir tablodaki veriye daha hızlı erişebilmek için oluşturulan özel veri yapısıdır.

Yani:

> Kitapta bir konu ararken “içindekiler” kısmına bakmak gibidir 📘
> 
> — Index varsa doğrudan sayfayı bulursun,
> 
> — Yoksa kitabın tamamını taramak zorunda kalırsın.

---

## ⚙️ 2️⃣ Index Ne İşe Yarar?

Bir tabloyu sorguladığında (örneğin):

```sql
SELECT * FROM Employees WHERE FirsName = 'Ali';

```

- Eğer **index yoksa** → SQL tablonun **her satırını tek tek tarar** (table scan).
- Eğer **index varsa** → SQL “index ağacına” bakar, **doğrudan ilgili satırı bulur**.

Sonuç:

- **Arama hızlanır**
- **Okuma (SELECT)** performansı artar
- Ancak **yazma (INSERT / UPDATE / DELETE)** işlemleri biraz yavaşlayabilir (index de güncellenir çünkü).

---

## 💡 3️⃣ Index Türleri (SQL Server örnekleriyle)

|Tür|Açıklama|
|---|---|
|**Clustered Index**|Veriyi fiziksel olarak sıralar (tablo düzenini değiştirir). Her tabloda yalnızca **1 tane** olabilir.|
|**Non-Clustered Index**|Ek bir yapı oluşturur (kitap sonundaki fihrist gibi). Tablodan ayrı tutulur, birden fazla olabilir.|

---

## 🧱 4️⃣ Örnek Tablomuz

```sql
CREATE TABLE Employees
(
    Id INT PRIMARY KEY, -- bu zaten clustered index oluşturur
    FirsName NVARCHAR(50),
    City NVARCHAR(50),
    Age INT
);

```

🧠 Not:

- `PRIMARY KEY` otomatik olarak **clustered index** oluşturur.
- Yani tablo Id’ye göre fiziksel olarak sıralanır.

---

## 📈 5️⃣ Non-Clustered Index Oluşturma

Diyelim ki sık sık şehre göre arama yapıyoruz:

```sql
CREATE NONCLUSTERED INDEX IX_Employees_City
ON Employees (City);

```

### 🔍 Artık şu sorgu **çok daha hızlı** çalışır:

```sql
SELECT * FROM Employees WHERE City = 'Istanbul';

```

> SQL artık City kolonundaki indeks ağacına bakar,
> 
> satırı doğrudan bulur — tüm tabloyu taramaz.

---

## 🔍 6️⃣ Birden Fazla Kolonla Index

```sql
CREATE NONCLUSTERED INDEX IX_Employees_City_Age
ON Employees (City, Age);

```

Bu durumda şu sorgu da hızlanır:

```sql
SELECT * FROM Employees WHERE City = 'Ankara' AND Age = 30;

```

---

## ⚖️ 7️⃣ Index Kullanım Dengelemesi

|Durum|Index Kullanımı Uygun mu?|Neden|
|---|---|---|
|Çok sık **SELECT** var|✅ Evet|Okuma işlemleri hızlanır|
|Çok sık **INSERT / UPDATE** var|⚠️ Dikkat|Her veri değişiminde index de güncellenir|
|Tablo küçük (az veri)|❌ Gereksiz|Index ek yük getirir|
|Tablo büyük (binlerce satır)|✅ Çok faydalı|Arama farkı dramatiktir|

---

## ⚙️ 8️⃣ Index Nasıl Görülür / Silinir

### 📜 Var olan indexleri görmek:

```sql
EXEC sp_helpindex 'Employees';

```

### ❌ Index silmek:

```sql
DROP INDEX IX_Employees_City ON Employees;

```

---

## 🧠 9️⃣ Clustered vs Non-Clustered Farkı (görsel benzetme)

|Özellik|Clustered Index|Non-Clustered Index|
|---|---|---|
|Veri sıralaması|Fiziksel olarak sıralı|Ayrı bir yapıdadır|
|Sayı|1 tane olabilir|Birden fazla olabilir|
|Otomatik oluşan|Primary Key|Manuel oluşturulur|
|Erişim hızı|Çok hızlı|Hızlı ama dolaylı|
|Depolama|Tabloda tutulur|Ayrı alanda tutulur|

---

## 🔬 10️⃣ Örnek: Farkı Gerçekten Hissetmek

### 1️⃣ Büyük tablo oluşturalım

```sql
CREATE TABLE TestData
(
    Id INT IDENTITY PRIMARY KEY,
    Name NVARCHAR(50),
    Age INT
);

DECLARE @i INT = 1;
WHILE @i <= 50000
BEGIN
    INSERT INTO TestData (Name, Age)
    VALUES ('Ali' + CAST(@i AS NVARCHAR(10)), (RAND() * 100));
    SET @i = @i + 1;
END;

```

### 2️⃣ Index olmadan sorgu

```sql
SELECT * FROM TestData WHERE Age = 50;

```

➡ Çok yavaş olur (çünkü tüm tabloyu tarar).

### 3️⃣ Index ekle:

```sql
CREATE NONCLUSTERED INDEX IX_TestData_Age
ON TestData (Age);

```

### 4️⃣ Tekrar sorgu:

```sql
SELECT * FROM TestData WHERE Age = 50;

```

➡ Gözle görülür fark: çok daha hızlı ⚡

---

## ⚠️ 11️⃣ Index’in Yan Etkileri

|Durum|Etki|
|---|---|
|`INSERT`|Index güncellenir → az da olsa yavaşlatır|
|`UPDATE`|Indexteki kolon değişirse güncellenir|
|`DELETE`|Indexteki kayıt da silinir|
|Çok fazla index|Diskte fazla yer kaplar, veri güncellemelerini yavaşlatır|

💡 Bu yüzden:

> “En çok kullanılan kolonlara” index koy,
> 
> “her kolona” değil ❗

---

## ✅ 12️⃣ Mülakat için kısa özet (not defterine yazmalık)

> 🔹 Index, veritabanında veriye hızlı erişim sağlar.
> 
> 🔹 Clustered Index veriyi fiziksel olarak sıralar (bir tane olur).
> 
> 🔹 Non-Clustered Index ek bir yapı oluşturur (birden fazla olabilir).
> 
> 🔹 SELECT hızlanır, ama INSERT/UPDATE işlemleri biraz yavaşlar.
> 
> 🔹 Gereksiz index performansı düşürür, dikkatli kullanılmalıdır.