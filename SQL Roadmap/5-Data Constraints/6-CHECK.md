

`CHECK`, veritabanına girilecek verilerin senin belirlediğin **özel kurallara** (Business Logic) uyup uymadığını denetleyen kısıtlamadır. Veritabanının kapısındaki güvenlik görevlisidir; kurallara uymayan veriyi içeri almaz.

**Neden Kullanılır?** `NOT NULL` sadece boş mu dolu mu diye bakar. `UNIQUE` sadece tekrar var mı diye bakar. Ama "Ürün fiyatı negatif olamaz" veya "Bitiş tarihi başlangıç tarihinden küçük olamaz" gibi mantıksal kuralları denetlemek için `CHECK` kısıtlamasına ihtiyacımız vardır.

---

### Deep Dive: Veri Kalitesinin Sigortası

Bir Junior Developer genelde validasyonları (doğrulamaları) C# tarafında yapmayı sever. Ama mülakatta "Validasyonu neden sadece kodda değil de veritabanında da yapmalıyız?" sorusu gelirse, cevabın `CHECK` kısıtlaması olacaktır.

#### 1. Kural Çeşitleri ve Kullanımı

SQL'in mantıksal operatörlerini kullanarak neredeyse her türlü kuralı yazabilirsin:

- **Aralık Kontrolü:** `CHECK (Age >= 18 AND Age <= 120)` -> Yaş 18 ile 120 arasında olmalı.
    
- **Negatiflik Kontrolü:** `CHECK (Price > 0)` -> Fiyat 0 veya eksi olamaz.
    
- **Liste Kontrolü:** `CHECK (Status IN ('Aktif', 'Pasif', 'Beklemede'))` -> Bu üç kelime dışında bir durum yazılamaz. (Enum mantığı).
    
- **Format Kontrolü:** `CHECK (Email LIKE '%@%.%')` -> Basit e-posta formatı kontrolü.
    

#### 2. Sütunlar Arası İlişki - _En Faydalı Özellik_

CHECK sadece tek bir sütuna bakmak zorunda değildir; aynı satırdaki iki sütunu birbiriyle kıyaslayabilir.

- **Senaryo:** İzin tablosunda `BaslangicTarihi` ve `BitisTarihi` var.
    
- **Kural:** Bitiş tarihi, başlangıçtan önce olamaz.
    
- **SQL:** `CHECK (BitisTarihi >= BaslangicTarihi)`
    
- Bu kuralı koymazsan, bir gün raporlarda "eksi 5 gün izin kullanan" personel görürsün.
    

#### 3. NULL Tuzağı (Unknown Logic) - _Mülakat Sorusu_

CHECK kısıtlaması, kuralın sonucu **FALSE** ise kaydı reddeder. Ancak **NULL** (Unknown) durumunda ne yapar?

- **Kural:** `CHECK (Age > 18)`
    
- **Senaryo:** `Age` alanına `NULL` gönderdin.
    
- **Mantık:** `NULL > 18` işleminin sonucu `UNKNOWN`'dur (Bilinmiyor). Bilinmeyen şey `FALSE` değildir.
    
- **Sonuç:** SQL Server, CHECK kısıtlamalarında `UNKNOWN` sonucunu **kabul eder**. Yani `CHECK` kısıtlaması `NULL` verileri engellemez (Bunun için `NOT NULL` kullanmalısın). Bu detay mülakatlarda çok can yakar.
    

---

### Backend Developer İçin Neden Önemli?

1. **Defense in Depth (Derinlemesine Savunma):** "Ben zaten C# kodunda `if (price < 0) throw new Exception()` yazdım, veritabanına gerek yok" deme.
    
    - Ya birisi veritabanına kodunu bypass edip direkt Management Studio'dan bağlanıp veri eklerse?
        
    - Ya başka bir mikroservis senin veritabanına yazarken validasyonu atlarsa?
        
    - Veritabanı en alt katmandır (Last Line of Defense). Veri bütünlüğü için kurallar orada da olmalıdır.
        
2. **Entity Framework Core:** EF Core'da `ModelBuilder` ile bu kısıtlamaları ekleyebilirsin:
    
    C#
    
    ```csharp
    modelBuilder.Entity<Product>()
        .ToTable(t => t.HasCheckConstraint("CK_Price_Positive", "Price > 0"));
    ```
    
    Bu sayede Code-First yaklaşımında bile SQL tarafında `CHECK` constraint oluşur.
    
3. **Hata Yönetimi:** Kullanıcı negatif fiyat girdiğinde veritabanı `SqlException` fırlatır. Bu hatayı yakalayıp kullanıcıya "Geçersiz fiyat girdiniz" diyebilirsin.
    

CHECK Constraint, verinin mantıksal tutarlılığı için kritikti. Özellikle "Sütunlar arası kıyaslama" ve "NULL davranışı" detaylarını not ettiysen harika.