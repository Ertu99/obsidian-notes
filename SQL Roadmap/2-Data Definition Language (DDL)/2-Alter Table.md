`ALTER TABLE`, mevcut bir veritabanı tablosunun yapısını (structure), içindeki verileri silmeden değiştirmek için kullanılan DDL komutudur.

**Neden Kullanılır?** Yazılım projeleri yaşayan organizmalardır; ihtiyaçlar sürekli değişir. Başta "Kullanıcı adı 50 karakter yeter" dersin, 6 ay sonra "100 karakter olsun" denir. Ya da "TC Kimlik No" sütununu eklemeyi unutmuşsundur. Tabloyu silip baştan oluşturmak (DROP & CREATE) içindeki verileri yok edeceği için, `ALTER TABLE` ile tadilat yaparız.

---

### Deep Dive: Tadilatın Riskleri ve Teknik Detaylar

Bir Junior Developer olarak Localhost'ta çalışırken `ALTER TABLE` çok masum görünür. Ancak **Canlı (Production)** ortamda, içi milyonlarca veriyle dolu bir tabloda bu komutu çalıştırmak kalp çarpıntısı yaratır. Mülakatta bu riskleri bildiğini göstermelisin.

#### 1. Temel Operasyonlar

- **ADD (Sütun Ekleme):** Tabloya yeni bir alan ekler.
    
    - _Komut:_ `ALTER TABLE Users ADD PhoneNumber VARCHAR(20)`
        
    - _Dikkat:_ Yeni eklenen sütun, eski kayıtlar için `NULL` olarak gelir. Eğer `NOT NULL` olarak eklersen ve varsayılan değer (Default Value) vermezsen hata alırsın.
        
- **DROP COLUMN (Sütun Silme):** Bir sütunu ve içindeki tüm veriyi siler.
    
    - _Komut:_ `ALTER TABLE Users DROP COLUMN Age`
        
    - _Risk:_ Geri dönüşü yoktur. Veri kaybıdır.
        
- **ALTER/MODIFY COLUMN (Tip Değiştirme):** Bir sütunun veri tipini veya boyutunu değiştirir.
    
    - _Komut:_ `ALTER TABLE Users ALTER COLUMN Name VARCHAR(100)`
        

#### 2. Veri Kaybı Riski (Data Truncation) - _Mülakat Sorusu_

Sütun boyutunu **büyütmek** (`VARCHAR(50)` -> `VARCHAR(100)`) genelde sorunsuzdur. Ancak **küçültmek** (`VARCHAR(100)` -> `VARCHAR(10)`) tehlikelidir.

- Eğer tabloda 50 karakterlik bir isim varsa ve sen sütunu 10 karaktere düşürürsen, SQL Server ya hata verir ya da veriyi keser (Truncate). "Ali Veli Maria..." -> "Ali Veli M" olur.
    

#### 3. Tip Dönüşümü (Type Conversion)

Bir sütunu `VARCHAR`'dan `INT`'e çevirmek istersen (Örn: "123" -> 123), içindeki verilerin hepsi sayısal olmalıdır. Eğer arada bir tane bile "ABC" varsa işlem patlar.

#### 4. Table Locking (Tablo Kilitleme) - _En Büyük Tehlike_

Mülakatta sorarlarsa +100 puan: _"Canlı sistemde ALTER TABLE yaparken neye dikkat edersin?"_

- **Cevap:** `ALTER TABLE`, işlemin büyüklüğüne göre tabloyu bir süreliğine **kilitler (Lock)**.
    
- Milyonlarca satırlı bir tabloda, `INT` olan bir kolonu `BIGINT` yapmak tüm tabloyu yeniden yazmayı (Rewrite) gerektirebilir. Bu işlem 10 dakika sürerse, 10 dakika boyunca o tabloya kimse erişemez, site "Donar" veya "TimeOut" hatası verir. Bu yüzden bu işlemler gece yarısı veya bakım modunda yapılır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core Migrations:** Sen C# kodunda `public string Name { get; set; }` özelliğini değiştirip `Add-Migration` dediğinde, EF Core senin için arka planda bir `ALTER TABLE` scripti hazırlar. `Update-Database` dediğinde bu script veritabanına uygulanır.
    
2. **Validasyon (Doğrulama):** Eğer veritabanındaki sütun `VARCHAR(50)` ise ve sen Backend tarafında (C# veya FluentValidation ile) 50 karakter sınırını kontrol etmezsen, kullanıcı 60 karakter gönderdiğinde uygulama veritabanı hatası (`String or binary data would be truncated`) fırlatır ve patlar. Veritabanı yapısıyla kodunun uyumlu olması şarttır.