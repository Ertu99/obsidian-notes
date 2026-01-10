Normalde `GROUP BY` kullandığında detay satırlarını kaybedersin, sadece toplamı görürsün. Window Functions ise sana şunu der: **"Hem pastam dursun hem karnım doysun."** Yani hem detay satırlarını gör hem de yanına özet (toplam, ortalama, sıra) bilgisini ekle.

Mantığını ve en çok kullanacağın 3 türünü inceleyelim:

---

### Temel Sözdizimi: `OVER()`

Window Function'ın imzası `OVER` kelimesidir. Nerede `OVER` görürsen orada bir pencere açılmış demektir.

SQL

```sql
FonksiyonAdi() OVER (
    PARTITION BY [Gruplama Sütunu] -- Pencereyi neye göre böleceğiz? (Opsiyonel)
    ORDER BY [Sıralama Sütunu]     -- Pencere içinde sıralama nasıl olacak?
)
```

---

### 1. Sıralama Fonksiyonları (Ranking)

Mülakatların vazgeçilmezidir. _"Her departmandaki en yüksek maaş alan personeli bul"_ sorusunun cevabıdır.

- **`ROW_NUMBER()`:** Satırlara 1, 2, 3 diye eşsiz numara verir. Eşitlik olsa bile (Maaşlar aynı) rastgele birine 1, diğerine 2 der.
    
- **`RANK()`:** Olimpiyat madalyası gibidir. İki kişi 2. olursa, 3. atlanır, sıradaki 4. olur (1, 2, 2, 4).
    
- **`DENSE_RANK()`:** Boşluk bırakmaz. İki kişi 2. olursa, sıradaki 3. olur (1, 2, 2, 3).
    

**Örnek:** Her departmanda (`PARTITION BY Department`) maaşa göre sıralama yap.

SQL

```sql
SELECT
    Name,
    Department,
    Salary,
    ROW_NUMBER() OVER(PARTITION BY Department ORDER BY Salary DESC) as SiraNo
FROM Employees;
```

_Bu sorgu sonucunda her departmanın kendi içinde 1'den başlayıp sıralandığını görürsün._

### 2. Öteleme Fonksiyonları (Offset - LAG/LEAD)

Veri analitiğinde "Bir önceki aya göre değişim" hesaplamak için kullanılır. Eskiden bunu yapmak için tabloyu kendisiyle (`SELF JOIN`) birleştirirdik, çok yavaştı.

- **`LAG(Sütun, N)`:** Şu anki satırdan N satır **gerideki** veriyi getirir.
    
- **`LEAD(Sütun, N)`:** Şu anki satırdan N satır **ilerideki** veriyi getirir.
    

**Senaryo:** Bu ayki satışı, geçen ayki satışla yan yana görmek.

SQL

```sql
SELECT
    Ay,
    Satis,
    LAG(Satis, 1) OVER(ORDER BY Ay) as GecenAySatis
FROM Satislar;
```

### 3. Kümülatif Toplam (Running Total)

Banka hesap dökümü gibi, her satırda bakiyenin üzerine ekleyerek gitmek. Normal `SUM` fonksiyonunu `OVER(ORDER BY ...)` ile kullanırsan kümülatif çalışır.

SQL

```sql
SELECT
    IslemTarihi,
    Tutar,
    SUM(Tutar) OVER(ORDER BY IslemTarihi) as GuncelBakiye
FROM HesapHareketleri;
```

---

### Backend Developer İçin Neden Önemli?

1. **Duplicate (Tekrar Eden) Kayıtları Silmek:** Bir tabloda yanlışlıkla aynı veriden 2-3 tane oluştu. Bunları nasıl temizlersin?
    
    - `ROW_NUMBER()` ile gruplayıp numaralandırırsın.
        
    - `DELETE FROM CTE WHERE RowNum > 1` dersin. Bu, en temiz yöntemdir.
        
2. **Sayfalama (Pagination):** Modern SQL'de `OFFSET/FETCH` var ama eski sistemlerde veya karmaşık sıralamalarda `ROW_NUMBER()` ile sayfalama yapılır ("Sayfa 2'yi getir" demek "Sıra numarası 11 ile 20 arasını getir" demektir).
    
3. **Performans:** Karmaşık raporlar için veriyi C# tarafına çekip `for` döngüsüyle "önceki satırla karşılaştırma" yapmak CPU'yu yorar. `LAG/LEAD` bunu veritabanı motorunda (C++ hızında) yapar ve sana hazır sonucu verir.