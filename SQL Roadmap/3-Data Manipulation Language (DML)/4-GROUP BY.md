`GROUP BY`, veritabanındaki benzer verilere sahip satırları gruplayarak özet bilgiler (Summary) çıkarmamızı sağlayan SQL komutudur.

**Neden Kullanılır?** Patronun önüne 1 milyon satırlık satış listesini koyarsan hiçbir şey anlamaz. Ama "Bana şehirlere göre toplam satışları ver" dediğinde; İstanbul'daki tüm satışları tek satıra, Ankara'dakileri tek satıra indirip yanlarına toplam tutarı yazman gerekir. `GROUP BY`, bu "Daraltma/Özetleme" işlemini yapar. Genellikle `COUNT` (Say), `SUM` (Topla), `AVG` (Ortalama Al) gibi **Aggregate Functions (Toplama Fonksiyonları)** ile birlikte kullanılır.

---

### Deep Dive: Gruplamanın Altın Kuralları

Mülakatlarda `GROUP BY` ile ilgili sorulan en büyük teknik soru, hangi kolonların seçilip seçilemeyeceği kuralıdır. Junior developerların en çok hata aldığı yer burasıdır.

#### 1. "Daraltma" (Collapse) Mantığı

`GROUP BY` kullandığın anda, detay satırlar kaybolur.

- **Senaryo:** `Satislar` tablosunda `UrunAd`, `Kategori`, `Fiyat` sütunları var.
    
- **Sorgu:** `SELECT Kategori, SUM(Fiyat) FROM Satislar GROUP BY Kategori`
    
- **Sonuç:** SQL, aynı kategorideki (örn: Elektronik) 100 satırı alır, bunları "Elektronik" adında **tek bir satıra** ezer. Fiyatlarını da toplayıp yanına yazar. Artık o 100 satırın detayına (hangi müşteri aldı, saat kaçta aldı vb.) bu sorguda erişemezsin.
    

#### 2. SELECT Listesi Kuralı - _Mülakat Sorusu_

Eğer sorgunda `GROUP BY` varsa, `SELECT` kısmına **sadece** şunları yazabilirsin:

1. `GROUP BY` ifadesinde geçen sütunlar (Örn: `Kategori`).
    
2. Aggregate Fonksiyonları (Örn: `SUM(Fiyat)`, `COUNT(*)`).
    

- **Hatalı Sorgu:** `SELECT Kategori, UrunAd, SUM(Fiyat) FROM Satislar GROUP BY Kategori`
    
- **Hata:** SQL sana kızar: _"Kardeşim, Kategori'ye göre grupladım ama 'UrunAd'ı ne yapayım? Elektronik grubunda TV de var Telefon da. Hangisini göstereyim?"_
    
- **Kural:** Gruplanmayan bir sütunu (Aggregate olmadan) SELECT'e yazamazsın.
    

#### 3. HAVING ile Filtreleme - _WHERE vs HAVING Farkı (Part 2)_

Önceki konuda değinmiştik, burada pekiştirelim.

- **Soru:** "Toplam satışı 10.000 TL'den büyük olan kategorileri getir."
    
- **Yanlış:** `SELECT Kategori... WHERE SUM(Fiyat) > 10000` -> **ÇALIŞMAZ!** Çünkü `WHERE`, gruplama yapılmadan _önce_ çalışır. Henüz toplam hesaplanmamıştır.
    
- **Doğru:** `SELECT Kategori... GROUP BY Kategori HAVING SUM(Fiyat) > 10000`.
    
- `HAVING`, gruplanmış ve hesaplanmış veriyi filtreler.
    

#### 4. Çoklu Gruplama

Birden fazla sütuna göre de gruplayabilirsin.

- **Sorgu:** `GROUP BY Sehir, Ilce`
    
- **Mantık:** Önce Şehirlere göre ayırır (İstanbul, Ankara). Sonra İstanbul'un içindeki İlçeleri ayırır (Kadıköy, Beşiktaş).
    
- Sonuçta her "Şehir-İlçe" kombinasyonu için bir satır oluşur.
    

---

### Backend Developer İçin Neden Önemli?

1. **Dashboard ve Raporlama:** Admin panellerinde gördüğün "Bugün Kayıt Olan Kullanıcı Sayısı", "Aylık Toplam Ciro", "En Çok Satan 5 Ürün" gibi widget'ların hepsi arkada `GROUP BY` sorgusu çalıştırır.
    
2. **LINQ GroupBy:** C# tarafında: `var result = context.Sales.GroupBy(x => x.Category).Select(g => new { Category = g.Key, Total = g.Sum(x => x.Price) }).ToList();` Bu kod, EF Core tarafından birebir SQL `GROUP BY` sorgusuna çevrilir. LINQ'teki `Key` kavramı, SQL'deki gruplanan sütuna denk gelir.
    
3. **Client vs Server Evaluation:** Eğer `GROUP BY` işlemini veritabanında yapmazsan (yani tüm datayı C#'a çekip `IEnumerable` olarak RAM'de gruplarsan), milyonlarca satırı ağ üzerinden taşırsın. Veritabanı motorları (SQL Server), bu matematiksel işlemleri yapmak için optimize edilmiştir. İşi her zaman SQL'e (Server-side) yıkmalısın.