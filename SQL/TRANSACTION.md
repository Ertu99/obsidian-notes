## 🎯 1️⃣ Transaction Nedir?

> Transaction, bir veya birden fazla SQL komutunu tek bir bütün (atomik işlem) olarak çalıştıran yapıdır.
> 
> Yani bu komutlardan **hepsi başarılı olursa** işlem tamamlanır (**COMMIT**),
> 
> **biri bile hata verirse** tüm işlem geri alınır (**ROLLBACK**).

---

## 💡 Gerçek Hayat Benzetmesi:

Bankada para transferi düşün 👇

1️⃣ Ali’nin hesabından 100 TL eksiltiliyor

2️⃣ Ayşe’nin hesabına 100 TL ekleniyor

> Eğer 1. adım başarılı olur ama 2. adımda hata çıkarsa ne olur?
> 
> Para havada kalır.

Bunu önlemek için SQL **transaction** kullanır:

> “Ya her iki adım da başarılı olacak, ya da hiçbiri yapılmayacak.”

---

## ⚙️ 2️⃣ Temel Kullanım Yapısı

```sql
BEGIN TRANSACTION;

-- 1. işlem
UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;

-- 2. işlem
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

-- Her şey başarılıysa işlemi onayla
COMMIT TRANSACTION;

```

Eğer bir hata olursa geri almak için:

```sql
ROLLBACK TRANSACTION;

```

---

## ⚠️ 3️⃣ Hata Yönetimiyle Birlikte (Try-Catch)

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
    UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'Hata oluştu, işlem geri alındı.';
END CATCH;

```

🧠 Bu yapı **Stored Procedure’lerde** sıkça kullanılır.

Amaç: sistemin **tutarlılığını korumak** (consistency).

---

## 🔒 4️⃣ Transaction’ın Özellikleri (ACID Prensipleri)

|Harf|Özellik|Açıklama|
|---|---|---|
|**A**|Atomicity|İşlem ya tamamen olur ya hiç olmaz.|
|**C**|Consistency|Veri bütünlüğü korunur.|
|**I**|Isolation|Aynı anda çalışan işlemler birbirini bozmaz.|
|**D**|Durability|Commit olan işlem kalıcıdır, sistem çökse bile kaybolmaz.|

Bu 4 prensip = Transaction mantığının temeli 🔥

## 8️⃣ Transaction Ne Zaman Kullanılır?

|Durum|Kullanım|
|---|---|
|Birden fazla tablo aynı anda güncelleniyorsa|✅ Evet|
|Finansal işlemler (para, stok, sipariş)|✅ Kesinlikle|
|Log yazma gibi basit işlemler|❌ Gerek yok|
|Performans çok kritikse (okuma ağırlıklı)|⚠️ Dikkatli kullanılmalı|

---

## ✅ 9️⃣ Kısa Özet (Not Defterine Yazmalık)

> 🔹 Transaction, SQL’de birden fazla işlemi tek bir bütün haline getirir.
> 
> 🔹 Başarılı olursa `COMMIT`, hata olursa `ROLLBACK` yapılır.
> 
> 🔹 Atomicity, Consistency, Isolation, Durability (ACID) prensipleri geçerlidir.
> 
> 🔹 TRY-CATCH ile hata yönetimi yapılır.
> 
> 🔹 Özellikle para transferi, stok güncelleme, sipariş işlemlerinde kullanılır.