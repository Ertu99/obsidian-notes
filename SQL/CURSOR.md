## 🎯 1️⃣ Cursor Nedir?

> Cursor, bir sorgunun sonucunu satır satır gezmeye yarayan SQL yapısıdır.
> 
> Normalde SQL **set tabanlıdır** (yani tüm satırları aynı anda işler).
> 
> Ama bazen her satırda **tek tek işlem** yapmak istersin — işte o zaman Cursor kullanılır.

---

## 💡 Basit benzetme:

> “SELECT tüm satırları tek seferde getirir.”
> 
> Cursor ise “for döngüsü gibi, birer birer getirir.”

---

## ⚙️ 2️⃣ Cursor’un Temel Kullanım Aşamaları

|Adım|Açıklama|
|---|---|
|1️⃣ `DECLARE`|Cursor tanımlanır.|
|2️⃣ `OPEN`|Cursor açılır (hazırlanır).|
|3️⃣ `FETCH NEXT`|Sıradaki satır okunur.|
|4️⃣ `WHILE` döngüsü|Her satır için işlem yapılır.|
|5️⃣ `CLOSE`|Cursor kapatılır.|
|6️⃣ `DEALLOCATE`|Bellekten tamamen silinir.|

---

## 💻 3️⃣ Örnek: Her çalışanın adını tek tek yazdırmak

```sql
DECLARE @Name NVARCHAR(50);

DECLARE employee_cursor CURSOR FOR
SELECT FirsName FROM Employees;

OPEN employee_cursor;

FETCH NEXT FROM employee_cursor INTO @Name;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Çalışan: ' + @Name;
    FETCH NEXT FROM employee_cursor INTO @Name;
END

CLOSE employee_cursor;
DEALLOCATE employee_cursor;

```

### 🔍 Açıklama:

- `DECLARE ... CURSOR FOR` → Hangi sorgunun sonucunu gezeceğini belirtir.
- `OPEN` → Cursor’ı başlatır.
- `FETCH NEXT` → Bir sonraki satırı getirir.
- `@@FETCH_STATUS = 0` → Okunacak satır varsa devam eder.
- `CLOSE` ve `DEALLOCATE` → Temizlik (aksi halde belleği meşgul eder).

---

## 🧩 4️⃣ Orta Seviye Örnek: Maaşlara %10 zam yapalım

```sql
DECLARE @EmpId INT, @OldSalary DECIMAL(10,2);

DECLARE salary_cursor CURSOR FOR
SELECT Id, Salary FROM Employees;

OPEN salary_cursor;

FETCH NEXT FROM salary_cursor INTO @EmpId, @OldSalary;

WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE Employees
    SET Salary = @OldSalary * 1.10
    WHERE Id = @EmpId;

    FETCH NEXT FROM salary_cursor INTO @EmpId, @OldSalary;
END

CLOSE salary_cursor;
DEALLOCATE salary_cursor;

```

### 🧠 Ne olur?

- Cursor sırayla her çalışanın **Id** ve **Salary** değerini okur.
- Her biri için maaşı %10 artırır.
- Tüm satırlar bittiğinde Cursor kapanır.

---

## ⚙️ 5️⃣ `@@FETCH_STATUS` Nedir?

> @@FETCH_STATUS, FETCH işleminin başarılı olup olmadığını döndürür.
> 
> | Değer | Anlam |
> 
> |--------|--------|
> 
> | 0 | Başarılı (satır bulundu) |
> 
> | -1 | Satır yok |
> 
> | -2 | Hata |

Yani:

```sql
WHILE @@FETCH_STATUS = 0

```

demek → “Okuyacak satır varsa devam et.”

---

## 🧠 6️⃣ Cursor Ne Zaman Kullanılır?

|Uygun Olduğu Durum|Neden|
|---|---|
|Her satır için özel işlem gerekiyorsa|Satır satır kontrol sağlar|
|Güncelleme mantığı karmaşıksa|IF / CASE kullanmak kolay olur|
|Tek tek kontrol gerektiren durumlar|Örneğin; toplu e-posta gönderimi, özel loglama|

Ama 👇

|Kaçınılması Gereken Durum|Neden|
|---|---|
|Çok büyük tablolarda|Yavaş çalışır|
|Basit işlemlerde|Normal `UPDATE` veya `JOIN` daha hızlıdır|

---

## ⚖️ 7️⃣ Cursor Alternatifleri (daha performanslı)

Bazı durumlarda Cursor kullanmadan da aynı işi yapabilirsin:

|İşlem|Daha iyi alternatif|
|---|---|
|Maaşlara %10 zam|`UPDATE Employees SET Salary *= 1.1`|
|Belirli şarta göre işlem|`CASE WHEN` veya `MERGE`|
|Döngüye benzer işlem|WHILE döngüsü + geçici tablo|

Yani **Cursor**, “zorunlu” olmadıkça tercih edilmez.

---

## 🧩 8️⃣ Cursor Türleri (ileri seviye bilgi)

|Tür|Açıklama|
|---|---|
|**STATIC**|Veriyi kopyalar, tablo değişse bile etkilenmez.|
|**DYNAMIC**|Tablo değişirse Cursor güncel veriyi görür.|
|**FAST_FORWARD**|Sadece ileri gider, en hızlı Cursor’dır.|
|**KEYSET**|Id’leri sabit, veriler değişken.|

Örnek:

```sql
DECLARE employee_cursor CURSOR FAST_FORWARD
FOR SELECT Id, Salary FROM Employees;

```

---

## ✅ 9️⃣ Kısa Mülakat Özeti (Defterlik)

> 🔹 Cursor, sorgu sonucunu satır satır işlemek için kullanılan yapıdır.
> 
> 🔹 `DECLARE`, `OPEN`, `FETCH`, `CLOSE`, `DEALLOCATE` adımlarından oluşur.
> 
> 🔹 Döngü mantığında çalışır ve `@@FETCH_STATUS` ile kontrol edilir.
> 
> 🔹 Küçük veri setlerinde uygundur ama büyük tablolarda yavaş çalışır.
> 
> 🔹 Mümkünse toplu işlemler (`UPDATE`, `JOIN`, `CASE`) tercih edilmelidir.