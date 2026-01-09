Constraints (Kısıtlamalar), veritabanının kapıdaki güvenlik görevlileridir (Bouncer). "Spor ayakkabıyla giremezsin" veya "Damsız girilmez" gibi kuralları vardır. Eğer gelen veri bu kurallara uymazsa, veritabanı o veriyi içeri almaz ve **hata fırlatır**.

**Neden Kullanılır?** Backend kodunda (C#) validation yazsan bile, birisi veritabanına doğrudan bağlanıp elle veri girmeye çalışabilir veya kodunda bir açık olabilir. Constraints, verinin **son savunma hattıdır**.

En temel ve hayati 6 kısıtlamayı inceleyelim:

---

### 1. Temel Kısıtlamalar (The Big 6)

#### A. PRIMARY KEY (Birincil Anahtar)

Tablodaki her satırın **kimlik kartıdır (TC No)**.

- **Kural:** Benzersiz (Unique) olmalıdır ve asla boş (NULL) olamaz.
    
- **Örnek:** `UserId`, `OrderId`. Her tabloda mutlaka 1 tane olmalıdır.
    

#### B. FOREIGN KEY (Yabancı Anahtar)

İki tabloyu birbirine bağlayan köprüdür.

- **Kural:** "Buraya yazdığın değer, diğer tabloda mutlaka var olmalıdır."
    
- **Senaryo:** `Siparisler` tablosuna `MusteriId` ekliyorsun. Eğer olmayan bir müşteri ID'si (örn: 9999) yazmaya çalışırsan Foreign Key seni durdurur. "Böyle bir müşteri yok!" der.
    

#### C. NOT NULL (Boş Geçilemez)

Sütunun boş bırakılmasını yasaklar.

- **Kural:** Veri girmek zorunludur.
    
- **Örnek:** Kullanıcının `Email` adresi veya `Password` alanı boş olamaz.
    

#### D. UNIQUE (Benzersizlik)

Bir sütundaki verilerin tekrar etmesini engeller.

- **Kural:** Bu değerden içeride sadece 1 tane olabilir.
    
- **Örnek:** TC Kimlik No veya E-posta adresi. (Primary Key de Unique'tir ama Unique Constraint NULL değer kabul edebilir -genelde 1 tane-).
    

#### E. CHECK (Mantıksal Kontrol)

Verinin içeriğini bir şarta bağlar.

- **Kural:** Yazılımcının belirlediği kurala (`IF` mantığı) uymalıdır.
    
- **Örnek:** `CHECK (Age >= 18)` -> 18 yaşından küçükleri kaydetme. `CHECK (Price > 0)` -> Fiyat eksi olamaz.
    

#### F. DEFAULT (Varsayılan Değer)

Eğer veri gönderilmezse otomatik olarak atanacak değeri belirler.

- **Örnek:** `CreatedDate` sütununa veri yollanmazsa otomatik olarak şimdiki zamanı (`GETDATE()`) yazar.
    

---

### 2. Column Level vs Table Level (Kritik Ayrım)

Prompt'unda geçen bu ayrım, özellikle **Composite Key** (Birden fazla sütunun birleşerek anahtar olması) durumunda önemlidir.

- **Column Level (Sütun Seviyesi):** Kısıtlamayı sütunu tanımladığın satırın hemen yanına yazarsın.
    
    SQL
    
    ```sql
    CREATE TABLE Users (
        Id INT PRIMARY KEY, -- Column Level
        Email VARCHAR(100) UNIQUE
    );
    ```
    
- **Table Level (Tablo Seviyesi):** Tüm sütunları tanımladıktan sonra en altta kısıtlamaları yazarsın. **Birden fazla sütunu kapsayan** kısıtlamalar mecburen burada yazılır.
    
    SQL
    
    ```sql
    CREATE TABLE UserRoles (
        UserId INT,
        RoleId INT,
        -- Bir kullanıcı aynı role iki kere atanamasın (Composite Primary Key)
        CONSTRAINT PK_UserRoles PRIMARY KEY (UserId, RoleId) -- Table Level
    );
    ```
    

---

### Backend Developer İçin Neden Önemli?

1. **Hata Kodları (Exception Handling):** Backend tarafında `try-catch` bloğu kurarken, veritabanından dönen hataları yakalarsın.
    
    - Eğer `Unique Constraint Violation` hatası (Örn: Error 2627) gelirse, kullanıcıya "Bu e-posta adresi zaten kullanımda" mesajı gösterirsin.
        
    - Eğer `Check Constraint` hatası gelirse, "Girdiğiniz veriler kurallara uymuyor" dersin.
        
2. **Veri Tutarlılığı (Orphan Data):** `Foreign Key` kullanmazsan, `Users` tablosundan bir kullanıcıyı sildiğinde, `Orders` tablosunda o kullanıcının siparişleri sahipsiz (Orphan) kalır. Veritabanı çöplüğe döner. Foreign Key, "Siparişi olan kullanıcıyı silemezsin" diyerek sistemi korur.
    
3. **İş Mantığı Tekrarı:** Backend kodunda `if (age < 18)` kontrolü yapsan bile, veritabanına `CHECK (Age >= 18)` koymak "Emniyet Kemeri" gibidir. Kodunda bug olsa bile veritabanı hatalı veriyi reddeder.
    

Veri bütünlüğünü sağlayan "Constraints" konusu tamam. Burası veritabanı tasarımının (Database Design) kalbidir.