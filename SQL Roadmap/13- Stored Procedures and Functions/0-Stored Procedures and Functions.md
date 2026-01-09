**STORED PROCEDURES AND FUNCTIONS (Saklı Yordamlar ve Fonksiyonlar)**

SQL dünyasının "Programlama" (Procedural) kısmına hoş geldin. Şimdiye kadar hep sorgu yazdık (`SELECT`, `INSERT`). Artık **kod yazacağız**.

Bu iki yapı, SQL komutlarını bir paket haline getirip veritabanına kaydetmemizi sağlar. Backend tarafında (C#) yazdığın "Method" veya "Function"ların veritabanındaki karşılıklarıdır.

**Neden Kullanılır?** Her seferinde `SELECT * FROM Users WHERE Age > 18 AND City = 'Istanbul' ...` diye uzun uzun sorgu yazıp sunucuya göndermek yerine, bu mantığı `GetIstanbulUsers` adıyla veritabanına kaydedersin. Backend'den sadece bu ismi çağırırsın.

---

### Temel Ayrım (Mülakat Sorusu)

Bir Junior Developer olarak ikisini karıştırman çok olasıdır. Mülakatta _"Stored Procedure ile Function arasındaki fark nedir?"_ sorusu %90 gelir.

#### 1. Stored Procedures (SP - Saklı Yordamlar)

- **Görevi:** İş yapmak (Action). Veri eklemek, silmek, güncellemek veya karmaşık raporlar çekmek.
    
- **Yetenek:** İçinde `INSERT`, `UPDATE`, `DELETE` yapabilir.
    
- **Dönüş Değeri:** Bir değer dönmek zorunda değildir. İsterse tablo döner, isterse sayı döner, isterse hiçbir şey dönmez (C#'taki `void` metodlar gibidir).
    
- **Çağrılış:** `EXEC Sp_Adi` şeklinde çağrılır. `SELECT` veya `WHERE` içinde **kullanılamaz**.
    

#### 2. Functions (UDF - Kullanıcı Tanımlı Fonksiyonlar)

- **Görevi:** Hesaplama yapmak (Calculation).
    
- **Yetenek:** Veritabanının durumunu değiştiremez! Yani içinde genellikle `INSERT`, `UPDATE` veya `DELETE` **yapılamaz**. Sadece veriyi okur ve hesaplar.
    
- **Dönüş Değeri:** Mutlaka bir değer (Scalar) veya bir tablo (Table-Valued) dönmek **zorundadır**.
    
- **Çağrılış:** Sorgu içinde kullanılır. `SELECT dbo.Topla(5, 10)` veya `WHERE dbo.YasHesapla(DogumTarihi) > 18` gibi.
    

---

### Backend Developer İçin Neden Önemli?

1. **Güvenlik (SQL Injection Kalkanı):** Stored Procedure'ler parametre ile çalıştığı için (Parameterized Queries), dışarıdan gelen zararlı kodları (SQL Injection) etkisiz hale getirir.
    
    - _Kötü:_ `sql = "SELECT * FROM Users WHERE Ad = '" + txtAd + "'"` (Hacklenmeye açık).
        
    - _İyi:_ `db.Database.ExecuteSqlRaw("EXEC sp_UserBul @Ad", paramAd)` (Güvenli).
        
2. **Performans (Pre-compiled):** Sen normal bir SQL sorgusu gönderdiğinde, sunucu bunu her seferinde tekrar yorumlar (Compile). SP'ler ise **önceden derlenmiştir**. İlk çalışmada bir "Plan" oluşturur ve sonraki çağrılarda o planı kullanır. Bu yüzden çok daha hızlıdır.
    
3. **Ağ Trafiği (Network Traffic):** Backend sunucun ile Veritabanı sunucun arasında 100 satırlık bir SQL kodunu taşımak yerine, sadece `EXEC sp_Rapor` komutunu taşırsın. Trafik azalır, hız artar.
    
4. **Bakım (Maintenance):** Vergi hesaplama kuralı değişti diyelim.
    
    - _SP Yoksa:_ C# kodunu aç, değiştir, derle, sunucuya tekrar yükle (Deploy).
        
    - _SP Varsa:_ Sadece veritabanındaki SP'yi güncelle (`ALTER PROCEDURE`). C# koduna dokunmana gerek kalmaz.
        

Bu başlık altında SP ve Fonksiyonların nasıl oluşturulduğunu detaylandıracağız.