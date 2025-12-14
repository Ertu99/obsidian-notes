
Bunu şöyle düşün: Bir gece kulübünün kapısındaki güvenlik görevlisi (Bouncer) veritabanı kısıtlayıcısıdır. İçeriye kimin girebileceğine, kimin giremeyeceğine, kıyafet kuralına (Business Rule) o karar verir. Eğer güvenlik olmazsa, içerisi kısa sürede karışır.

Yazılımda **"Garbage In, Garbage Out" (GIGO - Çöp girerse çöp çıkar)** diye bir prensip vardır. Kısıtlayıcılar, içeriye "çöp" veri girmesini engelleyen son savunma hattıdır.

---

### 1. NOT NULL Constraint (Zorunluluk Kuralı)

En basit ama en hayati kuraldır. Bir sütunun boş (`NULL`) kalıp kalamayacağını belirler.

- **Mühendislik Açısı:** C# tarafında `int` (boş olamaz) ve `int?` (boş olabilir - Nullable) ayrımı neyse, veritabanında da `NOT NULL` o'dur.
    
- **Neden Önemli?** Eğer "Kullanıcı Adı" sütununa `NOT NULL` koymazsan, birisi sisteme isimsiz kayıt olabilir. Sonra `SELECT * FROM Users WHERE Name = 'Ahmet'` sorgusu patlamaz ama mantıksız verilerle uğraşırsın.
    
- **SQL:**
    
    SQL
    
    ```sql
    CREATE TABLE Users (
        Id INT PRIMARY KEY,
        Username NVARCHAR(50) NOT NULL -- Asla boş geçilemez!
    );
    ```
    

---

### 2. UNIQUE Constraint (Benzersizlik Kuralı)

Bir sütundaki verilerin tekrar etmesini engeller.

- **Primary Key'den Farkı Ne?** (Çok Sorulur)
    
    1. Bir tabloda sadece **1 tane** `PRIMARY KEY` olabilir. Ama istediğin kadar (yüzlerce) `UNIQUE` constraint olabilir.
        
    2. `PRIMARY KEY` asla `NULL` olamaz. `UNIQUE` alanlar (genellikle) `NULL` olabilir.
        
- **Kullanım:** TC Kimlik No, E-posta adresi, Telefon numarası. Bunlar PK değildir (çünkü sistem ID'sini kullanırız) ama yine de benzersiz olmalıdır.
    
- SQL Server Tuzağı (Mülakat Sorusu):
    
    Standart SQL'de, UNIQUE bir alana birden fazla NULL girebilirsin (çünkü null null'a eşit değildir). ANCAK, SQL Server'da UNIQUE constraint olan bir sütuna SADECE BİR TANE NULL girebilirsin. İkinci null'u girmeye çalışırsan hata alırsın. (Bu SQL Server'a özgü bir davranış biçimidir).
    

---

### 3. PRIMARY KEY (Birincil Anahtar)

Bu kural aslında iki kuralın birleşimidir: **NOT NULL + UNIQUE.**

- Mühendislik Detayı (Clustered Index):
    
    Sen bir sütuna PRIMARY KEY dediğin anda, SQL Server arka planda o sütun için otomatik olarak bir "Clustered Index" oluşturur. Yani veriyi diskte fiziksel olarak o ID sırasına göre dizer. Bu yüzden PK üzerinden arama yapmak ( WHERE ID = 5 ) ışık hızındadır.
    

---

### 4. FOREIGN KEY (Yabancı Anahtar ve Referans Bütünlüğü)

İlişkisel veritabanının kalbidir. Bir tablodaki verinin, başka bir tablodaki veriyle tutarlı olmasını sağlar.

- **Senaryo:** `Siparisler` tablosunda `MusteriID` var. Eğer `Musteriler` tablosunda olmayan bir ID'yi (örn: 9999) siparişlere eklemeye çalışırsan, FK kuralı devreye girer: _"Dur! Böyle bir müşteri yok, bu siparişi kime yazıyorsun?"_ der ve işlemi reddeder.
    

#### Kritik Ayar: `ON DELETE` Davranışları

Müşteri silinirse siparişlerine ne olacak? FK oluştururken bunu seçmelisin:

1. **NO ACTION (Varsayılan):** Müşteriyi silemezsin! Önce git siparişlerini sil, sonra müşteriyi sil der. (En güvenlisidir).
    
2. **CASCADE (Zincirleme Silme):** Müşteriyi silersen, veritabanı gider o müşteriye ait **tüm siparişleri de otomatik siler.** (Çok tehlikelidir, yanlışlıkla tüm geçmişi silebilirsin).
    
3. **SET NULL:** Müşteriyi siler, siparişlerin `MusteriID` kısmını `NULL` yapar. (Sipariş durur ama sahipsiz kalır).
    

---

### 5. CHECK Constraint (İş Mantığı Kuralı)

Veritabanına özel koşullar yazmanı sağlar. C# kodunda `if (age < 18)` yazmak yerine bunu veritabanına gömersin.

- **Kullanım:**
    
    - Maaş negatif olamaz (`Maas >= 0`).
        
    - Yaş 18'den küçük olamaz (`Yas >= 18`).
        
    - Durum sadece 'Aktif' veya 'Pasif' olabilir (`Status IN ('Aktif', 'Pasif')`).
        
- **SQL:**
    
    SQL
    
    ```sql
    CREATE TABLE Calisanlar (
        Id INT PRIMARY KEY,
        Maas DECIMAL(18,2),
        CONSTRAINT CHK_MaasPozitif CHECK (Maas >= 0) -- Kural burada
    );
    ```
    
- **Faydas:** Eğer bir DBA (Veritabanı Yöneticisi) yanlışlıkla elle negatif maaş girmeye çalışırsa, veritabanı onu da engeller. Sadece uygulamanı değil, veriyi her yerden korur.
    

---

### 6. DEFAULT Constraint (Varsayılan Değer)

Veri girilmediğinde ne yazılacağını belirler.

- **Kullanım:**
    
    - Kayıt tarihi (`CreatedDate`): Veri girilmezse o anki zamanı (`GETDATE()`) bas.
        
    - Aktiflik (`IsActive`): Veri girilmezse `1` (True) bas.
        

---

### Mühendislik Tartışması: Validation Nerede Olmalı? (App vs DB)

C# geliştiricileri genelde validasyonu kod tarafında (FluentValidation vb.) yapar. Veritabanına constraint koymaya üşenir. Bu yanlıştır.

**Savunma Derinliği (Defense in Depth):**

1. **Frontend (Javascript):** Kullanıcı deneyimi içindir. (Hacker bypass edebilir).
    
2. **Backend (C#):** Güvenlik ve İş Mantığı içindir. (Ama ya başka bir uygulama veritabanına bağlanırsa?).
    
3. **Database (Constraints):** **Son Kaledir.** Veritabanına doğrudan SQL Management Studio'dan bağlanan bir yazılımcı bile kural dışı veri girememelidir.
    

**Örnek:** Sen C#'ta "TCKN benzersiz olsun" kontrolü yazdın. Ama veritabanında `UNIQUE` constraint yok.

- İki kullanıcı milisaniye farkla aynı TCKN ile kayıt ol butonuna bastı (Race Condition).
    
- C# kodu ikisi için de "Veritabanında bu TCKN yok" dedi (çünkü okuduğunda yoktu).
    
- İkisi de `INSERT` gönderdi.
    
- **Sonuç:** Veritabanında aynı TCKN ile iki kayıt oluştu. Veri bozuldu.
    
- **Çözüm:** Veritabanında `UNIQUE` constraint olsaydı, ikincisi veritabanı seviyesinde reddedilir ve hata fırlatırdı. Veri korunurdu.
    

---

