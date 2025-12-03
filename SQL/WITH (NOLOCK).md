## 1️⃣ Temel Tanım

> WITH (NOLOCK) ifadesi, bir SELECT sorgusunun
> 
> veriyi **kilitlemeden (lock almadan)** okumasını sağlar.

Yani:

> “Ben sadece bakacağım, kimseyi bekletme.” 😎

Bu, özellikle **çok fazla okuma yapılan sistemlerde (raporlama, dashboard)** performansı artırmak için kullanılır.

---

## ⚙️ 2️⃣ Normalde ne olur (NOLOCK olmadan)

SQL Server, bir satır üzerinde **UPDATE, INSERT veya DELETE** işlemi yaparken

bu satırı **kilitler (lock)** ki aynı anda başka biri okumasın veya değiştirmesin.

Bu güvenli ama bazen **okuma sorguları bekler**.

Örneğin:

```sql
SELECT * FROM Orders;  -- Bekler çünkü başka bir transaction update yapıyor

```

---

## ⚡ 3️⃣ WITH (NOLOCK) ne yapar?

`WITH (NOLOCK)` bu beklemeyi kaldırır.

```sql
SELECT * FROM Orders WITH (NOLOCK);

```

Bu sorgu:

- Kilit (lock) almaz ✅
    
- Başka işlemlerin kilidini beklemez ✅
    
- Performans artar ⚡
    
    Ama…
    
- Veriyi **henüz commit edilmeden** (yani yarım) okuyabilir ❌
    

---

## ⚠️ 4️⃣ Tehlikesi: “Dirty Read” (Kirli Okuma)

**Dirty Read** = Henüz kaydedilmemiş (ROLLBACK olabilir) veriyi okumak.

Örnek senaryo:

```sql
-- Transaction 1
BEGIN TRAN
UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
-- (Henüz commit edilmedi)

```

```sql
-- Transaction 2
SELECT Balance FROM Accounts WITH (NOLOCK);

```

👀 Ne olur?

- Transaction 2, “Balance -100” değerini görebilir,
    
    ama Transaction 1 sonradan ROLLBACK yaparsa bu veri **gerçekte hiç olmamış olur**.
    
- Yani **yanlış veri** okumuş olursun.
    

---

## 🧠 5️⃣ `WITH (NOLOCK)` aslında neyin kısayolu?

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

```

Bu satırla aynı etkiyi yapar.

Yani **“commit edilmemiş verileri bile oku”** demektir.

---

## 📘 6️⃣ Kullanım Senaryoları

|Durum|Kullanım|
|---|---|
|✅ Raporlama, dashboard sorguları|Sadece okumak istiyorsan ve küçük tutarsızlık önemli değilse|
|✅ Çok yoğun sistemlerde (yüksek trafik)|Bekleme süresini azaltır|
|❌ Finansal işlemler|Kesinlikle kullanılmaz (veri yanlış olabilir)|
|❌ Transaction içinde|Tehlikelidir (veri tutarsızlığı riski)|

---

## 💻 7️⃣ Örnek Kullanım

### Normal sorgu

```sql
SELECT * FROM Employees;

```

### Kilit almadan (performanslı ama riskli)

```sql
SELECT * FROM Employees WITH (NOLOCK);

```

### Join içinde de kullanılabilir

```sql
SELECT e.FirsName, d.DepartmentName
FROM Employees e WITH (NOLOCK)
JOIN Departments d WITH (NOLOCK)
    ON e.DeptId = d.Id;

```

---

## ⚖️ 8️⃣ Artı ve Eksi Yönleri

|Artı|Eksi|
|---|---|
|✅ Kilitlenme (blocking) azalır|❌ Commit edilmemiş veriyi okuyabilir (dirty read)|
|✅ Performans artar|❌ Eksik veya iki kez okunan satır olabilir|
|✅ Bekleyen sorgular hızlanır|❌ Kritik verilerde güvenilmez sonuçlar üretir|

---

## ✅ 9️⃣ Kısa Özet (not defterine yazmalık)

> 🔹 WITH (NOLOCK) veriyi kilitlemeden okumayı sağlar.
> 
> 🔹 Performansı artırır ama **dirty read** riski taşır.
> 
> 🔹 Özellikle **raporlama sorgularında** güvenli,
> 
> ama **para, stok, sipariş** gibi kritik verilerde kullanılmamalıdır.
> 
> 🔹 Aslında `READ UNCOMMITTED` izolasyon seviyesinin kısa halidir.