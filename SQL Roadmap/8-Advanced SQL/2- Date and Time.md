Yazılım geliştirmede en çok baş ağrıtan konulardan biri "Zaman" yönetimidir. Saat dilimleri (Timezones), artık yıllar (Leap Years) ve format farklılıkları (GG/AA/YYYY vs AA/GG/YYYY) backend geliştiricilerin kâbusu olabilir. SQL'in bu konudaki yetenekleri hayat kurtarıcıdır.

Verdiğin başlıkları **Veri Tipleri** ve **Fonksiyonlar** olarak iki grupta inceleyelim.

---

### Bölüm 1: Veri Tipleri (Kapsayıcılar)

Veritabanında zamanı nasıl sakladığımız, o veriyi nasıl kullanacağımızı belirler.

#### 1. DATE (Sadece Tarih)

Saat, dakika veya saniye bilgisi içermez. Sadece "Hangi Gün?" sorusuna cevap verir.

- **Format:** `YYYY-MM-DD` (Genellikle).
    
- **Kullanım:** Doğum günü, İşe giriş tarihi, Resmi tatil günleri.
    
- **C# Karşılığı:** `DateTime` (Ancak saat kısmı hep 00:00:00 gelir).
    

#### 2. TIME (Sadece Saat)

Tarih bilgisi içermez. Sadece "Günün hangi saati?" sorusuna cevap verir.

- **Format:** `HH:MM:SS`.
    
- **Kullanım:** Ders programı (09:00'da başlar), Alarm saati, Mağaza açılış saati.
    
- **C# Karşılığı:** `TimeSpan`.
    

#### 3. TIMESTAMP (Tarih + Saat)

Hem takvim gününü hem de saati (bazen mikrosaniyeye kadar) tutar. "Tam olarak ne zaman oldu?" sorusunun cevabıdır.

- **Kullanım:** Sipariş anı, Log kayıtları, Son güncelleme tarihi.
    
- **Önemli Uyarı (SQL Server Farkı):** Eğer Microsoft SQL Server kullanıyorsan; oradaki `DATETIME` veya `DATETIME2` tipi, buradaki standart `TIMESTAMP` tanımına karşılık gelir. SQL Server'da `TIMESTAMP` adında başka bir veri tipi vardır ama o tarih tutmaz, versiyonlama için kullanılır (Binary). Bu isim karmaşasına dikkat!
    

---

### Bölüm 2: Fonksiyonlar (Araçlar)

Elimizdeki tarih verisini işlemek için bu iki fonksiyonu çok sık kullanırız.

#### 4. DATEPART (Parçalama / Ayıklama)

Bir tarih verisinin içinden sadece ihtiyacın olan parçayı cımbızla çeker.

- **Senaryo:** "Bana sadece 2023 yılında yapılan satışları getir." veya "Doğum günü Mart ayında olan personelleri bul."
    
- **Syntax:** `DATEPART(HangiParca, TarihSutunu)`
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Sipariş tarihi 2025-10-07 olsun.
    SELECT DATEPART(YEAR, OrderDate)  -- Çıktı: 2025
    SELECT DATEPART(MONTH, OrderDate) -- Çıktı: 10
    ```
    

#### 5. DATEADD (Ekleme / Çıkarma)

Tarih üzerinde matematik işlemi yapar. Geleceği veya geçmişi hesaplar.

- **Senaryo:** "Üyelik 30 gün sonra bitecek, bitiş tarihini hesapla" veya "Son 1 haftadaki siparişleri getir."
    
- **Syntax:** `DATEADD(HangiBirim, KacTane, HangiTarihe)`
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Bugüne 1 yıl ekle (Abonelik bitişi)
    SELECT DATEADD(YEAR, 1, GETDATE());
    
    -- Bugünden 7 gün çıkar (Geçmiş hafta raporu)
    -- Not: Eksi sayı vererek çıkarma yaparsın.
    SELECT DATEADD(DAY, -7, GETDATE());
    ```
    

---

### Backend Developer İçin Neden Önemli?

1. **UTC Kuralı (Altın Kural):** Global bir proje yapıyorsan, veritabanına tarihi her zaman **UTC (Evrensel Saat)** olarak kaydetmelisin (`GETUTCDATE()`).
    
    - Eğer yerel saatle kaydedersen; İstanbul'daki sunucuya New York'tan giren müşteri sipariş verdiğinde, sipariş saati gelecekte veya çok geçmişte görünebilir.
        
    - Backend'de UTC kaydedip, Frontend'de kullanıcının tarayıcı saatine göre göstermek en doğru yöntemdir.
        
2. **C# `DateTime.Now` vs SQL `GetDate()`:** Mülakatta sorarlar: _"Oluşturma tarihini (CreatedDate) C# kodunda mı atarsın (`DateTime.Now`), yoksa SQL'de default value (`GETDATE()`) olarak mı?"_
    
    - **Doğru Cevap:** Genellikle **SQL tarafında** (`GETDATE()` veya `CURRENT_TIMESTAMP`) yapılması tercih edilir. Çünkü uygulama sunucun ile veritabanı sunucun arasında saat farkı (milisaniyelik bile olsa) veya saat dilimi farkı olabilir. Veritabanı "Tek Gerçek Kaynak"tır (Single Source of Truth).
        
3. **LINQ Çevirisi:** C# kodunda `context.Orders.Where(o => o.Date > DateTime.Now.AddDays(-30))` yazdığında, EF Core bunu arka planda otomatik olarak `DATEADD` fonksiyonuna çevirir.