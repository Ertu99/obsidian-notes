## 1️⃣ TRY–CATCH nedir?

> SQL Server’daki TRY...CATCH,
> 
> komutlar çalışırken oluşabilecek **hataları yakalamak** ve
> 
> **işlemi güvenli şekilde yönetmek** için kullanılır.

Yani:

- `BEGIN TRY ... END TRY` → hatanın oluşabileceği alan
- `BEGIN CATCH ... END CATCH` → hata olursa çalışan alan

---

## 💻 2️⃣ Temel Söz Dizimi (Syntax)

```sql
BEGIN TRY
    -- Hata çıkma ihtimali olan SQL kodları
END TRY
BEGIN CATCH
    -- Hata olduğunda çalışacak SQL kodları
END CATCH

```

---

## ⚙️ 3️⃣ Basit Örnek

```sql
BEGIN TRY
    PRINT 'İşlem başlatıldı.';
    DECLARE @x INT = 10, @y INT = 0;
    SELECT @x / @y AS Sonuc;  -- Hata: sıfıra bölünme
    PRINT 'Bu satır çalışmaz çünkü hata oluştu.';
END TRY
BEGIN CATCH
    PRINT 'Bir hata oluştu!';
END CATCH;

```

### 🔍 Çıktı:

```
İşlem başlatıldı.
Bir hata oluştu!

```

Yani **TRY** bloğunda hata çıkınca SQL direkt **CATCH** bloğuna atlar,

ve oradaki kodlar çalışır.

---

## 🧠 4️⃣ CATCH içinde hata bilgisi nasıl alınır?

SQL Server hata detaylarını almak için özel fonksiyonlar sağlar 👇

|Fonksiyon|Açıklama|
|---|---|
|`ERROR_NUMBER()`|Hata numarası|
|`ERROR_MESSAGE()`|Hata mesajı|
|`ERROR_SEVERITY()`|Hatanın ciddiyet seviyesi|
|`ERROR_STATE()`|Hata durumu|
|`ERROR_LINE()`|Hatanın olduğu satır|
|`ERROR_PROCEDURE()`|Hatanın oluştuğu SP adı (varsa)|

### 🔍 Örnek:

```sql
BEGIN TRY
    DECLARE @x INT = 1 / 0;
END TRY
BEGIN CATCH
    PRINT 'Hata Numarası: ' + CAST(ERROR_NUMBER() AS NVARCHAR(10));
    PRINT 'Mesaj: ' + ERROR_MESSAGE();
    PRINT 'Satır: ' + CAST(ERROR_LINE() AS NVARCHAR(10));
END CATCH;

```

### 🧾 Çıktı:

```
Hata Numarası: 8134
Mesaj: Divide by zero error encountered.
Satır: 2

```

---

## 💵 5️⃣ TRY–CATCH + TRANSACTION birlikte kullanımı

Bu en yaygın kullanım şeklidir (özellikle **para transferi**, **güncelleme**, **silme** işlemlerinde).

### 💻 Örnek:

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
    UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

    COMMIT TRANSACTION;
    PRINT 'Transfer tamamlandı.';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'Hata oluştu: ' + ERROR_MESSAGE();
END CATCH;

```

### 🧠 Mantık:

- Eğer işlemlerin hepsi başarılı olursa → **COMMIT**
- Herhangi birinde hata olursa → **ROLLBACK** (geri al)
- Sistem yarım kalmaz, veri bütünlüğü korunur ✅

---

## ⚙️ 6️⃣ TRY–CATCH dışında SQL hata davranışı nasıldır?

Normalde SQL Server, hata oluştuğunda:

- O sorguyu durdurur ❌
- Ama sonraki sorguları çalıştırmaya devam eder.

TRY–CATCH bunu engeller:

> Hata çıkarsa, kontrolü CATCH bloğuna atar ve senin karar vermeni sağlar.

---

## 💬 7️⃣ TRY–CATCH bloklarında `THROW` kullanımı (hata fırlatma)

Bazı durumlarda hatayı **tekrar dışarı fırlatmak** istersin.

### 🔍 Örnek:

```sql
BEGIN TRY
    DECLARE @x INT = 1 / 0;
END TRY
BEGIN CATCH
    PRINT 'Hata yakalandı, şimdi dışarı fırlatıyorum.';
    THROW; -- hatayı tekrar gönder
END CATCH;

```

🧠 `THROW` → hata ayrıntısını korur (CATCH’te bastırmak yerine dışarı gönderir).

`RAISERROR`’ın modern halidir.

Örnek:

```sql
THROW 50001, 'Transfer hatası oluştu.', 1;

```

---

## ✅ 9️⃣ Kısa Özet (not defterine yazmalık)

> 🔹 TRY...CATCH SQL Server’da hataları yakalamak ve yönetmek için kullanılır.
> 
> 🔹 `BEGIN TRY` içinde hata olursa kontrol `BEGIN CATCH` bloğuna geçer.
> 
> 🔹 CATCH bloğunda hata detayları `ERROR_MESSAGE()`, `ERROR_NUMBER()` gibi fonksiyonlarla alınabilir.
> 
> 🔹 `TRY–CATCH` çoğunlukla `TRANSACTION` ile birlikte kullanılır (COMMIT / ROLLBACK).
> 
> 🔹 `THROW` veya `RAISERROR` ile hatalar tekrar fırlatılabilir.
> 
> 🔹 Bu yapı veri bütünlüğü ve güvenli hata yönetimi sağlar.