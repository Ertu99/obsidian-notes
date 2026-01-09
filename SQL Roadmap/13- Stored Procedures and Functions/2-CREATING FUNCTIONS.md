Stored Procedure'ler "İş Yapan" (Action) elemanlardı, Fonksiyonlar (User Defined Functions - UDF) ise veritabanının **"Hesap Makineleri"**dir.

Fonksiyonların altın kuralı şudur: **Veritabanındaki veriyi değiştiremezler.** Yani fonksiyon içinde `INSERT`, `UPDATE` veya `DELETE` yapamazsın. Sadece var olan veriyi alır, bir işlemden geçirir ve sonucu geri verir.

Fonksiyonlar iki ana türe ayrılır ve oluşturma mantıkları biraz farklıdır:

---

### 1. Scalar Functions (Tek Değer Döndürenler)

C#'taki `int Topla(int a, int b)` metodu gibidir. Geriye tek bir sayı veya metin (String, Int, Decimal) döner.

**Senaryo:** Ürünlerin fiyatına KDV ekleyip son fiyatı hesaplayan bir fonksiyon yazalım.

SQL

```sql
-- Fonksiyonlar her zaman bir şema (dbo) altına kaydedilir.
CREATE FUNCTION dbo.fn_KdvHesapla
(
    @Fiyat DECIMAL(18, 2),
    @Oran INT
)
RETURNS DECIMAL(18, 2) -- Geriye ne döneceğini baştan söylüyoruz
AS
BEGIN
    DECLARE @Sonuc DECIMAL(18, 2);

    SET @Sonuc = @Fiyat + (@Fiyat * @Oran / 100);

    RETURN @Sonuc; -- Hesaplanan değeri fırlat
END
```

**Nasıl Kullanılır?** Fonksiyonları `SELECT` içinde veya `WHERE` şartında kullanabilirsin.

- **Kritik Kural:** Çağırırken mutlaka şema adını (`dbo.`) yazmalısın!
    

SQL

```sql
SELECT
    UrunAdi,
    Fiyat,
    dbo.fn_KdvHesapla(Fiyat, 20) AS KdvliFiyat -- Fonksiyon burada çalışır
FROM Urunler;
```

---

### 2. Table-Valued Functions (Tablo Döndürenler)

Bu fonksiyonlar geriye tek bir değer değil, bildiğin **sanal bir tablo** (Liste) döner. "Parametre alabilen View" gibidirler.

**Senaryo:** Sadece belirli bir kategoriye ait ürünleri getiren fonksiyon.

SQL

```sql
CREATE FUNCTION dbo.fn_KategoriUrunleri
(
    @KategoriId INT
)
RETURNS TABLE -- Geriye tablo dönecek
AS
RETURN
(
    SELECT UrunAdi, Fiyat, Stok
    FROM Urunler
    WHERE KategoriId = @KategoriId
);
```

**Nasıl Kullanılır?** Geriye tablo döndüğü için, bunu `SELECT` listesinde değil, **`FROM`** kısmında kullanırsın.

SQL

```sql
-- Fonksiyonu sanki bir tablopmuş gibi sorguluyoruz
SELECT * FROM dbo.fn_KategoriUrunleri(5);
```

---

### Backend Developer İçin Kritik Uyarılar

#### 1. Performans Tuzağı (RBAR - Row By Agonizing Row)

Scalar fonksiyonlar (ilk örnek), büyük tablolarda çok tehlikelidir.

- Eğer 1 milyon satırlı bir tabloda `SELECT dbo.fn_KdvHesapla(Fiyat)...` dersen, SQL motoru o fonksiyonu 1 milyon kere tek tek çalıştırır. Sorgun kağnı hızına düşer.
    
- **Çözüm:** Basit matematiksel işlemleri fonksiyon yerine direkt sorgu içinde (`Fiyat * 1.20`) yapmak, performans açısından genelde daha iyidir.
    

#### 2. Kod Tekrarını Önleme (DRY Principle)

Çok karmaşık bir iş mantığın varsa (örneğin: Kıdem tazminatı hesaplama), bunu hem C# tarafında hem Python tarafında ayrı ayrı yazmak yerine, SQL'de bir Fonksiyon olarak yazmak mantıklıdır. Tek bir yerden güncellersin, tüm uygulamalar güncel veriyi kullanır.

#### 3. Computed Columns (Hesaplanmış Sütunlar)

Tablo oluştururken fonksiyonları kullanabilirsin.

- `CREATE TABLE Satislar (..., ToplamTutar AS dbo.fn_Carp(Adet, Fiyat))`
    
- Bu sayede veri girildiği an SQL otomatik hesaplama yapar.
    

Fonksiyonları; türleri ve kullanım alanlarıyla inceledik. SP'ler iş yapar, Fonksiyonlar hesap yapar.