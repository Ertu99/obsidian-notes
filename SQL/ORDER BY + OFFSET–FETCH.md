## 1️⃣ Genel Tanım

> OFFSET → kaç satır atlayacağını belirtir
> 
> `FETCH NEXT ... ROWS ONLY` → ardından **kaç satır alacağını** söyler

Bu yapı genellikle:

- Web sayfalarında “Sayfa 2, Sayfa 3” gibi verileri listelemek için,
- veya tabloyu parça parça okumak istediğinde kullanılır.

---

## 💻 2️⃣ Basit Örnek

Diyelim ki **Employees** tablon şu şekilde:

|Id|FirsName|Salary|
|---|---|---|
|1|Ali|8000|
|2|Ayşe|9000|
|3|Mehmet|9500|
|4|Elif|11000|
|5|Hasan|7000|
|6|Zeynep|7600|
|7|Veli|8200|
|8|Hakan|9500|
|9|Derya|10000|
|10|Murat|10500|
|11|Seda|7200|
|12|Kerem|8800|

---

## ⚙️ 3️⃣ Şimdi örnek sorguya bakalım

```sql
SELECT * FROM Employees
ORDER BY Id
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;

```

### 🔍 Anlamı:

- `ORDER BY Id` → Sıralama yapmadan OFFSET/FETCH çalışmaz
- `OFFSET 10 ROWS` → İlk **10 satırı atla**
- `FETCH NEXT 10 ROWS ONLY` → Sonraki **10 satırı getir**

Yani:

→ 1–10. satırları atlar

→ 11–20. satırları getirir ✅

---

## 🧩 4️⃣ 1. sayfa, 2. sayfa mantığı

|Sayfa|Sorgu|Açıklama|
|---|---|---|
|Sayfa 1|`OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY`|İlk 10 kayıt|
|Sayfa 2|`OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY`|11–20 arası|
|Sayfa 3|`OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY`|21–30 arası|

Bu yüzden genellikle uygulamalarda **sayfa numarasına göre** dinamik hesap yapılır 👇

```sql
DECLARE @PageNumber INT = 3;
DECLARE @PageSize INT = 10;

SELECT * FROM Employees
ORDER BY Id
OFFSET (@PageNumber - 1) * @PageSize ROWS
FETCH NEXT @PageSize ROWS ONLY;

```

💡 Burada `@PageNumber` değiştikçe, sayfa otomatik değişir.

---

## ⚠️ 5️⃣ Dikkat Edilmesi Gerekenler

|Nokta|Açıklama|
|---|---|
|`ORDER BY` zorunludur|OFFSET–FETCH sıralama olmadan çalışmaz|
|`OFFSET` 0 olabilir|“Hiç atlama, baştan başla” anlamına gelir|
|Performans|Büyük tablolarda OFFSET büyüdükçe yavaşlayabilir (örn. OFFSET 100000)|
|Alternatif|`ROW_NUMBER()` fonksiyonu da aynı amaçla kullanılabilir|

---

## 💡 6️⃣ Alternatif: `ROW_NUMBER()` ile aynı işlem

```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY Id) AS RowNum
    FROM Employees
) AS t
WHERE RowNum BETWEEN 11 AND 20;

```

Bu da 11–20 arası kayıtları getirir ama OFFSET/FETCH yerine klasik yöntemdir.

OFFSET–FETCH SQL Server 2012 ve sonrası için daha modern yöntemdir ✅

---

## ✅ 7️⃣ Kısa Özet (not defterine yazmalık)

> 🔹 OFFSET → kaç satırı atlayacağını belirtir.
> 
> 🔹 `FETCH NEXT ... ROWS ONLY` → ardından kaç satır alacağını belirtir.
> 
> 🔹 Genellikle sayfalama (pagination) işlemleri için kullanılır.
> 
> 🔹 `ORDER BY` olmadan çalışmaz.
> 
> 🔹 Örnek:
> 
> `OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY` → 11–20. kayıtları getirir.