`WHERE`, SQL sorgularında verileri **filtrelemek** için kullanılan koşul ifadesidir. Veritabanındaki tablolarda milyonlarca satır olabilir; `WHERE` bloğu sayesinde sadece belirlediğimiz kriterlere uyan (True dönen) satırların getirilmesini, güncellenmesini veya silinmesini sağlarız.

**Neden Kullanılır?** Eğer `WHERE` kullanmazsan; `SELECT` tüm tabloyu çeker (performans kaybı), `UPDATE` herkesin şifresini değiştirir (felaket), `DELETE` tüm veriyi siler (veri kaybı). Yani `WHERE`, işlem yapılacak verinin sınırlarını çizen güvenlik şerididir.

---

### Deep Dive: Filtrelemenin Teknik Derinlikleri

Bir Junior Developer olarak `WHERE` yazmak kolaydır (`ID = 5` dersin biter). Ancak mülakatta seni zorlayacakları kısım **Performans** ve **Çalışma Mantığı**dır.

#### 1. Çalışma Sırası (Execution Order) - _Tekrar Hatırlatma_

Önceki derste konuştuğumuz gibi; `WHERE` bloğu, veriler diskten (`FROM`) alındıktan hemen sonra, ancak kullanıcıya sunulmadan (`SELECT`) hemen önce çalışır.

- **Kural:** `SELECT` bloğunda verdiğin takma isimleri (Alias) `WHERE` bloğunda kullanamazsın.
    
    - _Yanlış:_ `SELECT Price * 1.18 AS KDVliFiyat FROM Products WHERE KDVliFiyat > 100`
        
    - _Hata:_ "Invalid column name 'KDVliFiyat'". Çünkü `WHERE` çalıştığında henüz o takma isim oluşmadı.
        
    - _Doğru:_ `WHERE (Price * 1.18) > 100`
        

#### 2. Performans ve Index Kullanımı (SARGable) - _Senior Seviyesi Bilgi_

Mülakatta _"WHERE bloğunda nelere dikkat edersin?"_ derlerse şu cevabı ver: **"Sorgunun SARGable (Search ARGument ABLE) olmasına dikkat ederim."**

Bu şu demek: `WHERE` bloğunda sütunun kendisine fonksiyon uygulamamalısın. Eğer uygularsan, SQL o sütundaki **Index'i kullanamaz** ve tüm tabloyu tek tek okur (Full Table Scan).

- **Kötü Örnek (Yavaş):** `WHERE YEAR(CreatedDate) = 2023` (SQL, her satırın tarihini alıp yılını hesaplamak zorundadır. Index devre dışı kalır).
    
- **İyi Örnek (Hızlı):** `WHERE CreatedDate >= '2023-01-01' AND CreatedDate <= '2023-12-31'` (SQL doğrudan tarih aralığına gider, Index kullanır).
    

#### 3. Implicit Conversion (Gizli Dönüştürme) Tuzağı

Veritabanında `PhoneNumber` sütunu `VARCHAR` (metin) olarak tanımlı olsun. Sen sorguda `WHERE PhoneNumber = 5321234567` (Sayı olarak) yazarsan:

- SQL, metin olan sütunu senin verdiğin sayıyla kıyaslamak için arka planda tüm sütunu sayıya çevirmeye çalışır.
    
- Bu hem performans kaybıdır hem de içinde harf olan bir kayıt varsa hata verir.
    
- **Doğrusu:** `WHERE PhoneNumber = '5321234567'` (Tırnak içinde).
    

#### 4. WHERE vs HAVING Farkı - _Klasik Soru_

İkisi de filtreleme yapar ama yerleri farklıdır.

- **WHERE:** Satırları gruplamadan **önce** filtreler. (Ham veriyi eler).
    
- **HAVING:** Satırları grupladıktan (`GROUP BY`) **sonra** filtreler. (Özet veriyi eler).
    
    - _Örnek:_ "İstanbul'daki kullanıcıları getir" -> `WHERE City = 'Istanbul'`
        
    - _Örnek:_ "Toplam sipariş tutarı 1000 TL'den fazla olan şehirleri getir" -> `HAVING SUM(Total) > 1000`
        

---

### Backend Developer İçin Neden Önemli?

1. **IQueryable vs IEnumerable (EF Core):** .NET mülakatlarının en önemli konusudur.
    
    - `db.Users.Where(u => u.Age > 18)` yazdığında bu bir `IQueryable` döner. Yani bu filtre henüz C# tarafında çalışmaz, SQL sorgusu olarak veritabanına gider (`WHERE Age > 18`). Veri filtrelenmiş olarak gelir. **Doğru olan budur.**
        
    - Eğer önce `.ToList()` deyip sonra `.Where(...)` dersen, tüm tabloyu RAM'e çeker ve filtreyi RAM'de yapar. Bu sistemi kilitler. `Where` koşulunu veritabanına göndermek (Server-side evaluation) kritiktir.
        
2. **Concurrency (Eşzamanlılık) Kontrolü:** Bir veriyi güncellerken, başkası değiştirmiş mi diye kontrol etmek için `WHERE` kullanılır. `UPDATE Products SET Price = 100 WHERE Id = 1 AND Version = 5` Eğer versiyon değişmişse (başkası güncellediyse) `WHERE` koşulu tutmaz ve güncelleme yapılmaz (0 rows affected). Bu sayede veri bütünlüğü korunur.
    
3. **Soft Delete:** Birçok sistemde veriler gerçekten silinmez, `IsDeleted = true` yapılır. Backend developer olarak yazdığın her sorguya (veya EF Core'daki Global Query Filter'a) otomatik olarak `WHERE IsDeleted = false` eklemen gerekir ki silinenler gelmesin.