**DELETE**

`DELETE`, veritabanı tablosundan bir veya birden fazla satırı **silmek** için kullanılan DML komutudur.

**Neden Kullanılır?** Yanlış girilen bir kaydı düzeltmek, eski logları temizlemek veya GDPR (Kişisel Verilerin Korunması) gereği üyelikten ayrılan bir kullanıcının verilerini yok etmek için kullanılır. `UPDATE` veriyi değiştirir, `DELETE` ise veriyi yok eder.

---

### Deep Dive: Silme İşleminin Maliyeti ve Riskleri

Bir Junior Developer için `DELETE` komutu basit görünse de, sistem üzerindeki etkisi büyüktür. Mülakatlarda özellikle "Performans" ve "Veri Bütünlüğü" üzerinden sorular gelir.

#### 1. Unutulan WHERE Faciası (Bölüm 2)

Tıpkı `UPDATE` komutunda olduğu gibi, `DELETE` komutunda da `WHERE` koşulu hayati önem taşır.

- **Hatalı:** `DELETE FROM Users`
    
    - **Sonuç:** Tablodaki **TÜM** kullanıcılar silinir. Tablo yapısı kalır ama içi bomboş olur.
        
- **Güvenli:** `DELETE FROM Users WHERE Id = 10`
    
    - Sadece ID'si 10 olan kullanıcı silinir.
        

#### 2. DELETE vs TRUNCATE (Mülakatın Zirve Sorusu)

Bu farkı adın gibi bilmelisin. İkisi de siler ama yöntemleri farklıdır.

- **DELETE (DML):**
    
    - Satırları **teker teker** siler.
        
    - Her silinen satır için Transaction Log dosyasına kayıt atar (Bu yüzden yavaştır).
        
    - **Trigger'ları tetikler.** (Örn: Kullanıcı silinince ona veda maili atan bir trigger varsa çalışır).
        
    - `WHERE` koşulu kullanılabilir.
        
- **TRUNCATE (DDL):**
    
    - Tablonun veri sayfalarını (Pages) toptan serbest bırakır.
        
    - Log tutmaz (çok az tutar), çok **hızlıdır**.
        
    - Trigger'ları **tetiklemez**.
        
    - `WHERE` koşulu **kullanılamaz** (Ya hepsi ya hiç).
        

#### 3. Foreign Key ve Cascade Delete

Bir tablo başka bir tabloyla ilişkiliyse silme işlemi karmaşıklaşır.

- **Senaryo:** `Siparisler` tablosunda ID'si 5 olan bir sipariş var. Bu siparişin sahibi `Musteriler` tablosundaki "Ahmet".
    
- **Sorun:** Eğer "Ahmet"i silmeye çalışırsan (`DELETE FROM Musteriler WHERE Ad = 'Ahmet'`), SQL hata verir: _"Bu müşterinin siparişleri var, önce onları silmelisin."_ (Referential Integrity).
    
- **Cascade Delete:** Eğer veritabanı tasarımında ilişkiyi **ON DELETE CASCADE** olarak kurduysan; Ahmet'i sildiğin anda SQL otomatik olarak Ahmet'e ait tüm siparişleri de siler. Bu çok güçlü ama tehlikeli bir özelliktir.
    

---

### Backend Developer İçin Neden Önemli?

1. **Hard Delete vs Soft Delete (Sektör Standardı):** Gerçek hayattaki projelerin %90'ında `DELETE` komutu **kullanılmaz**.
    
    - **Hard Delete:** SQL `DELETE` komutu ile veriyi fiziksel olarak yok etmektir.
        
    - **Soft Delete (Mantıksal Silme):** Tabloya `IsDeleted` (bit/bool) adında bir sütun eklenir. Kullanıcı "Hesabımı Sil" dediğinde biz arka planda `UPDATE Users SET IsDeleted = 1 WHERE Id = ...` yaparız.
        
    - **Neden?** Müşteri "Yanlışlıkla sildim geri getir" dediğinde veya yasal bir soruşturma olduğunda veriye ihtiyaç duyulur. Backend kodlarında tüm sorgularına otomatik olarak `WHERE IsDeleted = 0` eklenir.
        
2. **Entity Framework Core:**
    
    - `context.Remove(user)` dediğinde EF Core `DELETE` sorgusu üretir.
        
    - Ancak Soft Delete yapacaksan, entity'nin durumunu `Modified` yapıp `IsDeleted` alanını güncellersin.
        
3. **Geri Dönüş (Rollback):** `DELETE` bir DML işlemi olduğu için Transaction içinde yapılırsa geri alınabilir.
    
    SQL
    
    ```sql
    BEGIN TRANSACTION
    DELETE FROM Orders WHERE Date < '2023-01-01'
    -- Hata fark edilirse:
    ROLLBACK -- Her şey geri gelir.
    ```