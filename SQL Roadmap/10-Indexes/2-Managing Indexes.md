İndeks yönetimi, sadece `CREATE INDEX` yazıp bırakmak değildir. İndeksler canlı organizmalar gibidir; veritabanı büyüdükçe bozulurlar, parçalanırlar (fragmentation) ve bakıma ihtiyaç duyarlar. Ayrıca gereksiz indeksler veritabanı için bir "yük" haline gelir.

Bu süreci "Oluşturma", "Bakım" ve "Temizlik" olarak 3 aşamada inceleyelim:

---

### 1. Akıllı Oluşturma (Smart Creation)

Rastgele her sütuna indeks atılmaz. Stratejik davranmak gerekir.

- **Hangi Sütunlar?**
    
    - `WHERE`, `JOIN`, `ORDER BY` ve `GROUP BY` içinde sık kullanılan sütunlar.
        
    - Seçiciliği (Selectivity) yüksek olan sütunlar (Email, TC No gibi).
        
- **Composite Index (Birleşik İndeks) ve Sıralama:**
    
    - Birden fazla sütunu tek indekste topluyorsan **sıra hayati önem taşır**.
        
    - **Kural (Leftmost Prefix):** İndeksi `(Ad, Soyad)` olarak oluşturursan;
        
        - Sadece "Ad" araması yaparsan indeks çalışır.
            
        - "Ad ve Soyad" araması yaparsan indeks çalışır.
            
        - Sadece "Soyad" araması yaparsan **indeks çalışmaz!** (Çünkü indeks Ad'a göre sıralı başlar).
            

### 2. İndeks Bakımı (Maintenance): Rebuild vs Reorganize

Veri ekleyip sildikçe indeks sayfaları disk üzerinde dağılır. Buna **Fragmentation (Parçalanma)** denir. Kitabın sayfalarının kopup rastgele yerlere sıkıştırıldığını düşün; okumak zorlaşır.

- **Reorganize (Düzenle):**
    
    - Hafif bakımdır. Dağınık sayfaları yan yana getirir.
        
    - Sistem çalışırken (Online) yapılabilir, kilitleme yapmaz.
        
    - Genellikle parçalanma %5 - %30 arasındaysa önerilir.
        
- **Rebuild (Yeniden İnşa Et):**
    
    - Ağır bakımdır. İndeksi tamamen siler ve sıfırdan oluşturur.
        
    - Genellikle parçalanma %30'un üzerindeyse gereklidir.
        
    - **Risk:** İşlem sırasında tabloyu kilitleyebilir (Lock). Canlı sistemde yaparken dikkatli olunmalıdır (Enterprise sürümlerde Online Rebuild seçeneği vardır).
        

### 3. İndeks Temizliği (Cleanup)

Kullanılmayan indeksler, veritabanının "Pırasa Saçı" gibidir.

- **Zararı Nedir?** Sen tabloya her `INSERT` veya `UPDATE` yaptığında, SQL gidip o kullanılmayan indeksi de güncellemek zorundadır. Yazma hızını yavaşlatır, diskte yer kaplar.
    
- **Tespit:** SQL Server'da `dm_db_index_usage_stats` gibi sistem tablolarına bakarak "Hiç okunmamış ama binlerce kez güncellenmiş" indeksleri tespit edip `DROP INDEX` ile silmelisin.
    

---

### Backend Developer İçin Neden Önemli?

1. **Migration Dosyaları:** Bir Python/Django veya C#/EF Core projesinde indeksler genellikle Migration dosyaları içinde yönetilir.
    
    - Geliştirme aşamasında, hangi sorguların yavaş çalıştığını analiz edip migration dosyana uygun indeksi eklemelisin.
        
2. **Ölü Yatırım (Unused Indexes):** Projenin başında "Belki lazım olur" diye eklediğin indeksler, 1 yıl sonra sistemin yavaşlama sebebi olabilir. Ara sıra DBA (Database Administrator) şapkası takıp gereksizleri temizlemek performansı artırır.
    
3. **Include Columns (Kapsayan Sütunlar):** Bazen `SELECT` listesindeki veriyi almak için ana tabloya gitmek (Key Lookup) maliyetlidir.
    
    - `CREATE INDEX IX_Email ON Users(Email) INCLUDE (Name, Surname)`
        
    - Bu indeks, Email'e göre sıralıdır ama yanında Ad ve Soyad verisini de taşır. Böylece sorgu sadece indeksi okuyup cevabı döner, tabloya hiç uğramaz. Buna **Covering Index** denir ve çok hızlıdır.
        

İndeks yönetimi, veritabanı performansının bel kemiğidir. "Aç-Kapa" değil, sürekli izlenmesi gereken bir süreçtir.