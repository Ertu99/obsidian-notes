`NOT NULL`, bir sütunun asla **boş (NULL)** değer alamayacağını garanti eden kısıtlamadır. Veritabanı dilinde "Bu alan zorunludur, doldurmadan geçemezsin" demektir.

**Neden Kullanılır?** Bir kayıt formunda yanlarında kırmızı yıldız (*) olan kutucuklar vardır ya; işte onların veritabanındaki karşılığı budur. Sistemin çalışması için hayati olan verilerin (Örn: Kullanıcı Adı, Şifre, TC No) eksik girilmesini engeller.

---

### Deep Dive: NULL Kavramı ve "Boşluk" Tuzağı

Bir Junior Developer olarak şu farkı netleştirmelisin: **NULL, sıfır veya boşluk demek değildir.**

#### 1. NULL Nedir?

- **NULL:** "Bilinmiyor", "Değer yok", "Henüz girilmedi" demektir. Hafızada (Memory) yeri yoktur (veya özel bir işaretçidir).
    
- **0 (Sıfır):** Bir sayıdır, bir değerdir.
    
- **'' (Empty String):** İçi boş bir metindir, bir değerdir. Hafızada yer kaplar.
    

#### 2. Mülakat Sorusu: Empty String vs NULL

Sana şunu sorarlar: _"Bir sütunu NOT NULL yaptım ama kullanıcı yine de boş veri kaydedebiliyor. Neden?"_

- **Cevap:** Çünkü kullanıcı (veya kod) veritabanına `NULL` değil, `''` (Empty String / Boş Tırnak) gönderiyordur.
    
- `NOT NULL` kısıtlaması `''` (boş metin) değerini kabul eder! Çünkü boş metin de bir veridir.
    
- Eğer boş metni de engellemek istiyorsan, ekstra olarak `CHECK (Len(Column) > 0)` gibi bir kural daha eklemelisin.
    

#### 3. Sonradan NOT NULL Ekleme (Production Riski)

İçi dolu, canlı bir tabloya sonradan `NOT NULL` bir sütun eklemek istersen ne olur?

- **Komut:** `ALTER TABLE Users ADD PhoneNumber VARCHAR(20) NOT NULL`
    
- **Hata:** SQL Server işlemi reddeder.
    
- **Neden:** Çünkü tabloda eski kayıtlar var. Yeni sütun eklenince eski kayıtların o sütunu ne olacak? SQL onları `NULL` yapmak ister ama sen `NOT NULL` dedin. Çelişki oluşur.
    
- **Çözüm:** Önce sütunu `DEFAULT` bir değerle (Örn: 'Bilinmiyor' veya 0) eklemelisin ki eski kayıtların içi dolsun.
    

---

### Backend Developer İçin Neden Önemli?

1. **C# Nullable Types:**
    
    - SQL'de `NOT NULL` olan bir `INT` sütunu, C#'ta `int` (Değer tipi) olarak karşılanır.
        
    - SQL'de `NULL` olabilir (`NULLABLE`) olan bir sütun, C#'ta `int?` (Nullable int) olarak karşılanır.
        
    - Eğer bu eşleşmeyi yanlış yaparsan, veritabanından veri çekerken `NullReferenceException` veya `InvalidCastException` alırsın.
        
2. **Validasyon Maliyeti:** Eğer veritabanında alanı `NOT NULL` yaptıysan, Backend kodunda (FluentValidation veya DataAnnotations ile) o alanın dolu olduğunu kontrol etmek **zorundasın**. Kontrol etmezsen ve boş veri gönderirsen, veritabanı hata fırlatır (`SqlException`). Veritabanı hatası yakalamak, kod içinde validasyon yapmaktan daha maliyetlidir (Performance).
    
3. **Required Attribute:** Entity Framework Core kullanırken modelin tepesine `[Required]` yazarsan, Migration aldığında bu veritabanına `NOT NULL` olarak yansır.
    

`NOT NULL` konusu, özellikle "Empty String" farkı ve C# tipleriyle ilişkisi açısından kritikti.