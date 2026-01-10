CTE, SQL dünyasının **"Clean Code"** prensibidir. Karmaşık, iç içe geçmiş (Nested) ve okunması imkansız hale gelen sorguları; düzenli, okunabilir ve yönetilebilir parçalara bölmemizi sağlar.

**Mantığı Nedir?** Backend tarafında (Python/C#) uzun bir fonksiyon yazarken nasıl aralarda değişkenler (`var totalSales = ...`) tanımlayıp aşağıda kullanıyorsan; CTE de SQL içinde geçici bir sanal tablo (değişken) tanımlayıp, hemen altındaki sorguda kullanmanı sağlar.

CTE'leri iki ana kategoride inceleriz: **Standart** ve **Recursive (Özyinelemeli)**.

---

### 1. Standart CTE (Okunabilirlik İçin)

İç içe `SELECT` sorguları (Subqueries) bir süre sonra "Spagetti Kod"a dönüşür. CTE bunu çözer.

**Senaryo:** Yıllık satış ortalamasının üzerinde satış yapan personeli bulalım.

- **CTE Olmadan (Kötü):**
    
    SQL
    
    ```sql
    SELECT * FROM Personel
    WHERE SatisTutari > (SELECT AVG(SatisTutari) FROM Satislar WHERE Yil = 2023)
    -- Aynı alt sorguyu başka yerde lazım olsa tekrar yazmak zorundasın.
    ```
    
- **CTE İle (Temiz):**
    
    SQL
    
    ```sql
    WITH OrtalamaSatis AS ( -- Sanal tabloyu tanımla
        SELECT AVG(SatisTutari) as Ort FROM Satislar WHERE Yil = 2023
    )
    SELECT * FROM Personel p
    JOIN OrtalamaSatis os ON p.SatisTutari > os.Ort; -- Aşağıda tertemiz kullan
    ```
    

### 2. Recursive CTE (Hiyerarşi İçin - Advanced)

CTE'nin asıl süper gücü buradadır. Kendi kendini çağıran sorgular yazmanı sağlar.

**Kullanım Alanı:** Ağaç yapısı (Tree Structure) olan veriler.

- E-Ticaret kategori ağacı (Elektronik -> Bilgisayar -> Laptop).
    
- Şirket Organizasyon Şeması (CEO -> Müdür -> Şef -> Personel).
    
- Forum yorumları (Yorum -> Alt Yorum -> Cevap).
    

**Nasıl Çalışır?** 3 parçadan oluşur:

1. **Anchor Member (Çapa):** Başlangıç noktasını belirler (Örn: CEO).
    
2. **Recursive Member (Döngü):** Kendini tekrar çağıran kısım (Örn: Bu kişinin altındakileri bul).
    
3. **Termination Check:** Döngü ne zaman bitecek? (Altında çalışan kalmayınca).
    

**Örnek (Kategori Ağacı):**

SQL

```sql
WITH KategoriAgaci AS (
    -- 1. Çapa: En üst kategorileri (ParentId IS NULL) bul
    SELECT Id, Ad, ParentId, 0 AS Seviye
    FROM Kategoriler
    WHERE ParentId IS NULL

    UNION ALL

    -- 2. Rekürsif: Bir üstteki sonuca göre altındakileri bul
    SELECT k.Id, k.Ad, k.ParentId, ka.Seviye + 1
    FROM Kategoriler k
    INNER JOIN KategoriAgaci ka ON k.ParentId = ka.Id -- Kendi kendini joinliyor!
)
SELECT * FROM KategoriAgaci;
```

Bu sorgu, kaç derinlikte kategori olursa olsun hepsini hiyerarşik sırayla getirir. Bunu `WHILE` döngüsüyle yapmaya çalışmak tam bir kâbustur.

---

### Backend Developer İçin Neden Önemli?

1. **Koddan Lojik Çıkarma:** Kategori ağacını C# tarafında `Recursion` fonksiyonu ile kurmak mümkündür ama veriyi çekerken 100 kere veritabanına gitmen (N+1) gerekebilir. Recursive CTE ile tek sorguda tüm ağacı çekip, C# tarafında sadece ekrana basarsın. Performans farkı devasadır.
    
2. **Geçici Tablo Alternatifi:** `#TempTable` oluşturmak bazen disk I/O maliyeti yaratır. CTE ise (genellikle) sadece hafızada (Memory) yaşar ve sorgu bitince yok olur. Daha hafiftir.
    
3. **View Yerine Kullanım:** Sadece tek bir raporda kullanacağın karmaşık bir lojik için veritabanında kalıcı bir `VIEW` oluşturup kirlilik yaratmak istemezsin. CTE, "Kullan-At View" gibidir.
    

**Not (Performans):** SQL Server'da CTE'ler (Recursive olmayanlar) performans artışı sağlamaz, sadece kodu güzelleştirir. Ancak PostgreSQL'de `MATERIALIZED` keyword'ü ile sonucu önbelleğe alıp performansı artırabilirsin.