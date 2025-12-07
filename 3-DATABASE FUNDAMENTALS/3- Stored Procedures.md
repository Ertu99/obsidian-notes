
---

### 1. Stored Procedure Nedir? (Analoji ve Gerçek)

En basit tabiriyle Stored Procedure (SP), veritabanı içinde saklanan bir **C# Metodu (Function)** gibidir.

- Parametre alır (Input).
    
- İçeride `IF-ELSE`, `WHILE` döngüsü, değişken tanımlama gibi programlama işleri yapar.
    
- Sonuç döner (Tablo, tek bir değer veya sadece başarı mesajı).
    

Normalde sen SQL sorgusunu C# tarafında string olarak (`"SELECT * FROM..."`) yazıp gönderirsin (Ad-Hoc Query). SP'de ise sorgu zaten veritabanında yazılıdır, sen sadece adını çağırırsın.

---

### 2. "Pre-compiled" Ne Demek? (Performansın Sırrı)

Metinde geçen "Stored procedures can improve database performance" cümlesinin altını açalım. Neden daha hızlı?

Sen veritabanına herhangi bir sorgu gönderdiğinde (ister SP olsun ister normal SQL), Veritabanı Motoru (Engine) şu 3 adımı izler:

1. **Parsing (Ayrıştırma):** Sözdizimi doğru mu? (`SELECK` mi yazdın `SELECT` mi?).
    
2. **Resolving (Çözümleme):** "Users" tablosu gerçekten var mı? "Age" sütunu orada mı?
    
3. **Optimization (Optimizasyon - En Kritik Adım):** Veriye gitmek için en hızlı yol (Execution Plan) hangisi? Index mi kullanayım, tabloyu mu tarayayım?
    

**Fark Şurada:**

- **Normal SQL:** Her gönderdiğinde bu 3 adım (özellikle maliyetli olan 3. adım) tekrar yapılır.
    
- **Stored Procedure:** SP ilk kez oluşturulduğunda veya ilk kez çalıştırıldığında, motor bu planı çıkarır ve **önbelleğe (Cache) atar.** İkinci kez çağırdığında plan hazırdır. Direkt çalışır. Bu, saniyede binlerce işlem alan sistemlerde devasa fark yaratır.
    

---

### 3. Güvenlik Mimarisi (Security Abstraction)

Metinde geçen "allowing database administrators to control which users have access" kısmı çok kritiktir.

Büyük kurumsal firmalarda (Banka, Telekom) güvenlik şu prensiple işler: **"Least Privilege" (En Az Yetki).**

- Yazılımcı olarak senin (veya uygulamanın kullanıcı adının) `Users` tablosuna `SELECT` veya `DELETE` atma yetkisi **yoktur.**
    
- Eğer tabloya doğrudan erişimin yoksa, veriyi nasıl çekeceksin?
    
- DBA (Database Admin) sana bir SP yazar: `sp_UserGetir`.
    
- Sana sadece bu SP'yi `EXECUTE` etme yetkisi verir.
    

Böylece sen sadece o SP'nin izin verdiği kurallar çerçevesinde veriye dokunabilirsin. "Yanlışlıkla tüm tabloyu sildim" diyemezsin çünkü `DELETE` yetkin yok, sadece `sp_UserGetir` yetkin var. Bu, **SQL Injection** saldırılarına karşı da çok güçlü bir kalkandır.

---

### 4. Kod Anatomisi (T-SQL Örneği)

Bir SP'nin iç yapısını, parametre alımını ve transaction yönetimini gösteren gerçekçi bir örnek:

SQL

```sql
CREATE PROCEDURE sp_ParaTransferi
    @GonderenID INT,
    @AliciID INT,
    @Miktar DECIMAL(18, 2)
AS
BEGIN
    -- Performans için gereksiz "1 row affected" mesajlarını kapatır
    SET NOCOUNT ON;

    -- Hata yönetimi için Transaction başlatıyoruz
    BEGIN TRY
        BEGIN TRANSACTION;

            -- 1. Para Çek
            UPDATE Hesaplar SET Bakiye = Bakiye - @Miktar 
            WHERE HesapID = @GonderenID;

            -- 2. Para Yatır
            UPDATE Hesaplar SET Bakiye = Bakiye + @Miktar 
            WHERE HesapID = @AliciID;

            -- 3. Loglama (İşlem kaydı)
            INSERT INTO IslemLoglari (Gonderen, Alici, Tutar, Tarih)
            VALUES (@GonderenID, @AliciID, @Miktar, GETDATE());

        COMMIT TRANSACTION; -- Her şey yolundaysa onayla
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION; -- Hata varsa her şeyi geri al (Atomicity)
        -- Hatayı fırlat ki C# tarafı bilsin
        THROW; 
    END CATCH
END
```

---

### 5. Mühendislik Tartışması: Business Logic Nerede Durmalı?

Bu konu sektörde bir savaştır. SP kullanmanın dezavantajlarını da bilmelisin:

1. **Vendor Lock-in (Bağımlılık):** SP'leri SQL Server'ın diliyle (T-SQL) yazarsın. Yarın şirket "PostgreSQL'e geçiyoruz" derse, PostgreSQL'in dili (PL/pgSQL) farklı olduğu için **tüm kodlarını çöpe atıp baştan yazman gerekir.** C# kodu ise veritabanından bağımsızdır (ORM kullanıyorsan).
    
2. **Debugging ve Test:** C# kodunu Visual Studio'da satır satır debug etmek, Unit Test yazmak çok kolaydır. SP'leri debug etmek ve test etmek daha zahmetli ve zordur.
    
3. **Versiyon Kontrolü (Git):** C# dosyaları Git'te harika yönetilir. Veritabanı kodlarını versiyonlamak ve merge conflict çözmek daha karmaşıktır.
    

**Modern Yaklaşım:** Genellikle karmaşık iş mantığı (Business Logic) C# katmanında tutulur, veritabanı sadece veri saklamak için kullanılır. Ancak **Raporlama** veya **Toplu Veri Güncelleme (Batch Jobs)** gibi performansın kritik olduğu yerlerde hala SP kraldır.

---

### 6. İleri Seviye Konu: Parameter Sniffing (Parametre Koklama)

Bu, SP'lerin baş belasıdır ve mülakatlarda "Senior" adaylara sorulur.

- **Olay:** SP ilk çalıştığında `Şehir = 'İstanbul'` (1 milyon kayıt) parametresiyle çağrıldı diyelim. SQL Engine buna uygun bir plan (Index Scan) yapar ve cache'ler.
    
- **Sorun:** İkinci kez `Şehir = 'Köy'` (3 kayıt) parametresiyle çağrıldığında, Engine hafızadaki "İstanbul Planı"nı kullanır. 3 kayıt için koca tabloyu tarar! Performans yerle bir olur.
    
- **Çözüm:** SP içinde `OPTION (RECOMPILE)` kullanarak "Her seferinde planı yeniden yap" diyebilirsin (ama bu da cache avantajını yok eder) veya local değişken kullanarak Engine'i kandırabilirsin.
    

---

### (Auto-Commit Modu)

SQL Server (ve çoğu modern RDBMS) varsayılan olarak **"Auto-Commit"** modunda çalışır. Yani her satır kendi başına bir işlemdir.

- SP içinde açıkça `BEGIN TRANSACTION` demezsen, veritabanı motoru şöyle davranır:
    
    1. Satır 10: `UPDATE ...` (Başarılı mı? Tamam, diske yaz ve kalıcı yap.)
        
    2. Satır 11: `UPDATE ...` (Hata mı verdi? Sadece bu satırı iptal et.)
        

Sonuç: Para gönderenden düştü (kalıcı oldu), alıcıya gitmedi (hata verdi). Para buharlaştı. Veri bütünlüğü (Consistency) bozuldu.

Bu yüzden altın kural şudur: **Birden fazla `INSERT/UPDATE/DELETE` işlemi içeren her SP, mutlaka `BEGIN TRANSACTION` ... `COMMIT/ROLLBACK` blokları içine alınmalıdır.**