İndeksler, SQL performans optimizasyonunun **en güçlü silahıdır**. Ancak bu silahı yanlış kullanırsan (gereksiz indeksleme), kendi ayağına sıkarsın (yavaşlayan yazma işlemleri).

Performans optimizasyonu başlığı altında indeksleri incelerken, odaklanmamız gereken nokta "İndeks nedir?" değil, **"İndeksleri nasıl stratejik kullanırız?"** sorusudur.

İndeks kullanım stratejilerini 3 ana maddede inceleyelim:

---

### 1. Doğru Adayları Belirlemek (Seçicilik)

Her sütuna indeks atılmaz. İndeksin gücü **"Selectivity" (Seçicilik)** oranına bağlıdır.

- **Yüksek Seçicilik (High Selectivity):** Bir sütundaki verilerin çoğu birbirinden farklıdır. (TC No, Email, Telefon, Sipariş No).
    
    - _Strateji:_ **Kesinlikle indeksle.** SQL motoru indeksi kullanarak milyonlarca kayıt arasından aradığını saniyeler içinde bulur ("Index Seek").
        
- **Düşük Seçicilik (Low Selectivity):** Veriler sürekli tekrar eder. (Cinsiyet: E/K, Durum: Aktif/Pasif).
    
    - _Strateji:_ **Genellikle indeksleme.** Eğer tablonun %50'si "Erkek" ise, SQL motoru indeksi okumakla uğraşmaz, gidip tabloyu taramayı (Scan) tercih eder. İndeks ölü yatırım olur.
        

### 2. Clustered vs. Non-Clustered (Kritik Mimari)

Veritabanı motorunun veriyi fiziksel olarak nasıl sakladığını anlaman gerekir.

- **Clustered Index (Kümelenmiş):**
    
    - Kitabın kendisidir. Veriler disk üzerinde bu indekse göre **sıralı** durur.
        
    - Bir tabloda **sadece 1 tane** olabilir (Genellikle Primary Key).
        
    - Veriye en hızlı erişim yoludur.
        
- **Non-Clustered Index (Kümelenmemiş):**
    
    - Kitabın arkasındaki "İndeksler" bölümüdür. Konuyu bulursun, yanında sayfa numarası (Pointer) yazar, gidip o sayfayı açarsın.
        
    - Bir tabloda birden fazla olabilir (`Email`, `Username` vb. için).
        

**Optimization İpucu:** `WHERE` koşulunda en çok aradığın sütun (örneğin `CreatedDate`), eğer Primary Key değilse, ona bir _Non-Clustered Index_ eklemek sorguyu uçurur.

### 3. Composite Indexes (Birleşik İndeksler) ve Sıralama

Birden fazla sütunu tek bir indekste toplamak (`CREATE INDEX IX_AdSoyad ON Users(Ad, Soyad)`).

- **Altın Kural (Leftmost Prefix):** İndeks `(A, B, C)` sırasıyla oluşturulduysa;
    
    - `WHERE A = ?` -> İndeks çalışır.
        
    - `WHERE A = ? AND B = ?` -> İndeks çalışır.
        
    - `WHERE B = ?` -> **İndeks ÇALIŞMAZ!** (Çünkü sıralama A ile başlıyor).
        
- **Strateji:** Sorgularını analiz et (`Query Analysis`). Eğer kullanıcılar hep "Ad ve Soyad" ile arama yapıyorsa Composite Index kullan, tek tek indeks yapmaktan daha performanslıdır.
    

---

### Backend Developer İçin Performans İpuçları

1. **Read vs Write Trade-off (Okuma/Yazma Dengesi):**
    
    - E-Ticaret sitesindeki `Products` tablosu %95 okunur, %5 yazılır. Buraya bol bol indeks koyabilirsin.
        
    - Sistemin `Logs` tablosu %99 yazılır (Insert), %1 okunur. Buraya indeks koyarken cimri olmalısın. Her indeks, `INSERT` işlemini yavaşlatır.
        
2. **Covering Index (Kapsayan İndeks):** Sorgun `SELECT Name, Surname FROM Users WHERE Email = '...'` ise; Sadece `Email` üzerinde indeks varsa, SQL önce indeksi bulur, sonra `Name` ve `Surname` verisini almak için ana tabloya gider (**Key Lookup** - Ekstra Maliyet).
    
    - _Çözüm:_ `CREATE INDEX IX_Email ON Users(Email) INCLUDE (Name, Surname)`
        
    - Böylece `Name` ve `Surname` verisi indeksin içine gömülür. SQL ana tabloya hiç gitmez, cevabı indeksten döner. I/O maliyeti düşer.
        
3. **İndeks Parçalanması (Fragmentation):** Sürekli veri silinip eklendikçe indeksler bozulur (defrag ihtiyacı). Backend tarafında "Database Maintenance Plan" oluşturup haftalık olarak indeksleri **Rebuild** veya **Reorganize** etmek performansı sürekli zirvede tutar.