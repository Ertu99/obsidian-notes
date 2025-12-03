# 🧩 SQL TRIGGER – Orta Seviye Anlatım ve Örnekler

## 🎯 1️⃣ Trigger Nedir?

> Trigger, bir tabloda INSERT, UPDATE veya DELETE işlemi gerçekleştiğinde
> 
> **otomatik olarak çalışan SQL kod bloğudur.**

Yani sen tabloya müdahale edince (veri eklendi, güncellendi veya silindiğinde)

SQL Server otomatik olarak bu Trigger’ı tetikler.

Sen çağırmazsın — sistem kendi çalıştırır. ⚙️

---

## ⚙️ 2️⃣ Trigger’ın Temel Yapısı (Sözdizimi)

```sql
CREATE TRIGGER trigger_adi
ON tablo_adi
AFTER INSERT / UPDATE / DELETE
AS
BEGIN
    -- Buraya işlem yapacak SQL kodları gelir
END;

```

### 🔹 Önemli:

- **AFTER** → işlem başarıyla tamamlandıktan sonra çalışır.
- **INSTEAD OF** → işlem yerine geçer (örneğin view’larda).

---

## 📋 3️⃣ Trigger Çalışma Mantığı

Trigger çalıştığında SQL Server **geçici (sanal)** iki tablo oluşturur:

|Tablo|Anlamı|
|---|---|
|`inserted`|Yeni gelen satır(lar) (INSERT’te eklenen, UPDATE’te yeni değerler)|
|`deleted`|Silinen satır(lar) (DELETE’te silinen, UPDATE’te eski değerler)|

Bu tablolar sayesinde, **önceki ve sonraki değerleri** karşılaştırabilirsin.

---

## 🧱 4️⃣ Orta Seviye Örnekler

---

### 🧩 A) AFTER INSERT – Yeni kayıt eklendiğinde log tutma

```sql
CREATE TABLE Employees
(
    Id INT PRIMARY KEY,
    FirsName NVARCHAR(50),
    Salary DECIMAL(10,2)
);

CREATE TABLE InsertLog
(
    LogId INT IDENTITY PRIMARY KEY,
    EmployeeId INT,
    Message NVARCHAR(100),
    InsertDate DATETIME DEFAULT GETDATE()
);

GO
CREATE TRIGGER trg_Employees_Insert
ON Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO InsertLog (EmployeeId, Message)
    SELECT i.Id, 'Yeni bir çalışan eklendi.'
    FROM inserted i;
END;

```

### 🔍 Açıklama:

- `AFTER INSERT` → sadece yeni kayıt eklenince çalışır.
- `inserted` tablosu → yeni eklenen satırları içerir.
- Her yeni çalışanda `InsertLog` tablosuna bir mesaj eklenir.

---

### 🧩 B) AFTER UPDATE – Maaş değişikliklerini kaydetme (değişim geçmişi)

```sql
CREATE TABLE SalaryLog
(
    LogId INT IDENTITY PRIMARY KEY,
    EmployeeId INT,
    OldSalary DECIMAL(10,2),
    NewSalary DECIMAL(10,2),
    ChangeDate DATETIME DEFAULT GETDATE()
);

GO
CREATE TRIGGER trg_Employees_SalaryUpdate
ON Employees
AFTER UPDATE
AS
BEGIN
    IF UPDATE(Salary)
    BEGIN
        INSERT INTO SalaryLog (EmployeeId, OldSalary, NewSalary)
        SELECT d.Id, d.Salary, i.Salary
        FROM deleted d
        JOIN inserted i ON d.Id = i.Id
        WHERE d.Salary <> i.Salary;
    END
END;

```

### 🔍 Açıklama:

- `IF UPDATE(Salary)` → sadece `Salary` kolonu değiştiyse çalışır.
- `deleted` = eski maaş, `inserted` = yeni maaş.
- Değişiklik varsa log tablosuna ekler.

---

### 🧩 C) AFTER DELETE – Silinen kayıtları arşive aktarma

```sql
CREATE TABLE EmployeeArchive
(
    Id INT,
    FirsName NVARCHAR(50),
    Salary DECIMAL(10,2),
    DeletedAt DATETIME DEFAULT GETDATE()
);

GO
CREATE TRIGGER trg_Employees_Delete
ON Employees
AFTER DELETE
AS
BEGIN
    INSERT INTO EmployeeArchive (Id, FirsName, Salary)
    SELECT d.Id, d.FirsName, d.Salary
    FROM deleted d;
END;

```

### 🔍 Açıklama:

- Silinen her çalışan bilgisi `EmployeeArchive` tablosuna kopyalanır.
- Böylece yanlışlıkla silinen verileri kurtarmak mümkündür (soft delete mantığı).

---

## 🔎 5️⃣ inserted / deleted Tablosu Mantığı

|İşlem Türü|inserted|deleted|Açıklama|
|---|---|---|---|
|INSERT|✅ Dolu|❌ Boş|Yeni kayıtlar burada|
|DELETE|❌ Boş|✅ Dolu|Silinen kayıtlar burada|
|UPDATE|✅ Dolu|✅ Dolu|Eski değer `deleted`, yeni değer `inserted`|

---

## 🧠 6️⃣ IF UPDATE(KolonAdi)

Trigger içinde hangi kolonun değiştiğini kontrol etmeni sağlar.

```sql
IF UPDATE(Salary)
BEGIN
    PRINT 'Maaş değişti!';
END;

```

---

## ⚖️ 7️⃣ Trigger’larda dikkat edilmesi gerekenler

|Dikkat Edilecek Nokta|Açıklama|
|---|---|
|**Performans**|Her DML (INSERT/UPDATE/DELETE) işlemine ekstra yük bindirir.|
|**Gizli çalışır**|Uygulama tarafından çağrılmaz, SQL otomatik tetikler.|
|**Çok karmaşık olmamalı**|Trigger içinde başka tabloyu güncellemek, yeniden tetiklemeye yol açabilir.|
|**Birden fazla satır**|`inserted` / `deleted` tablolarında _birden fazla satır_ olabilir; `JOIN` ile düşünmek gerekir.|
|**Recursive (tekrar tetikleme)**|Aynı tabloyu trigger içinde değiştirirsen kendini tekrar çalıştırabilir (sonsuz döngü).|

---

## ✅ 8️⃣ Kısa mülakat özeti (not defterine yazmalık)

> 🔹 Trigger, tabloya yapılan INSERT, UPDATE veya DELETE işlemlerinden sonra (veya yerine) otomatik olarak çalışan SQL kodudur.
> 
> 🔹 `inserted` tablosu yeni verileri, `deleted` tablosu eski verileri tutar.
> 
> 🔹 En çok log tutmak, arşivlemek, doğrulama yapmak veya tetikleme amacıyla kullanılır.
> 
> 🔹 Fazla kullanımı performans düşürür, bu yüzden dikkatli tasarlanmalıdır.