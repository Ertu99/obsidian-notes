SQL operatörleri, veritabanındaki veriler üzerinde filtreleme (süzme), matematiksel işlem yapma veya mantıksal karşılaştırmalar kurma işlemlerini sağlayan semboller veya anahtar kelimelerdir.

**Neden Kullanılır?** Veritabanındaki milyonlarca satır veriden sadece ihtiyacın olanları çekmek (WHERE koşulu), fiyatları KDV ile çarparak göstermek veya "Hem İstanbullu olsun Hem de 25 yaşından büyük olsun" gibi karmaşık sorgular yazmak için bu operatörlere muhtacız. Operatörler olmadan SQL sadece düz bir liste olurdu.

---

### Deep Dive: SQL Operatör Türleri ve Mülakat Tuzakları

Bu başlık basit görünür ama mülakatlarda "NULL kontrolü" veya "Performans farkları" ile ilgili tuzak sorular buradan gelir.

#### 1. Aritmetik Operatörler (Matematiksel)

Sayısal veriler üzerinde işlem yapar.

- `+`, `-`, `*`, `/`: Topla, Çıkar, Çarp, Böl.
    
- `%` (Modulo): Bölümünden kalanı verir.
    
    - _Kullanım:_ "Çift sayı ID'li kullanıcıları getir" (`ID % 2 = 0`) sorgusu için kullanılır.
        

#### 2. Karşılaştırma Operatörleri (Comparison)

İki değeri kıyaslar ve sonuç `TRUE` veya `FALSE` döner. `WHERE` bloğunda kullanılır.

- `=`: Eşittir.
    
- `<>` veya `!=`: Eşit Değildir. (Standart olan `<>`'dır ama `!=` de çalışır).
    
- `<`, `>`, `<=`, `>=`: Büyüktür, Küçüktür vb.
    

#### 3. Mantıksal Operatörler (Logical) - _Sorguların Omurgası_

Birden fazla koşulu bağlamak için kullanılır.

- **AND:** İki koşul da doğru olmalı.
    
- **OR:** Koşullardan en az biri doğru olsa yeter.
    
    - _Mülakat Sorusu (Öncelik Sırası):_ `WHERE A=1 OR B=2 AND C=3` yazdığında SQL önce `AND` işlemini yapar. Matematikteki "Çarpma toplamadan önce gelir" kuralı gibidir. Karışıklığı önlemek için **parantez `()`** kullanmak şarttır.
        
- **IN:** Bir değerin listede olup olmadığına bakar.
    
    - `WHERE City IN ('Istanbul', 'Ankara', 'Izmir')`. (Uzun uzun `OR` yazmaktan kurtarır).
        
- **BETWEEN:** İki değer aralığını seçer (Uç değerler dahildir).
    
    - `WHERE Price BETWEEN 100 AND 200`.
        
- **LIKE:** Metin içinde arama yapar (Pattern Matching).
    
    - `%` (Yüzde): Her şey anlamına gelir. `LIKE 'A%'` (A ile başlayanlar).
        
    - `_` (Alt çizgi): Tek bir karakter yerine geçer.
        
- **IS NULL / IS NOT NULL:** _En Büyük Mülakat Tuzağı!_
    
    - SQL'de `WHERE Age = NULL` diyemezsin! Çünkü NULL "bilinmeyen" demektir; bilinmeyen bir şey başka bir şeye eşit olamaz.
        
    - Doğrusu: `WHERE Age IS NULL`.
        

#### 4. Set (Küme) Operatörleri - _Mülakatın Yıldızı_

İki farklı SELECT sorgusunun sonucunu alt alta birleştirmek için kullanılır.

- **UNION vs UNION ALL (Klasik Soru):**
    
    - **UNION:** İki sorguyu birleştirir ama **tekrar eden kayıtları siler** (Duplicate removal). Ayrıca sonuçları arka planda sıralamaya çalışır. Bu yüzden **YAVAŞTIR**.
        
    - **UNION ALL:** İki sorguyu dümdüz alt alta yapıştırır. Tekrarlayanları silmez. İşlem yapmadığı için **HIZLIDIR**.
        
    - _Cevap:_ "Eğer verilerin benzersiz olduğundan eminsem performans için her zaman `UNION ALL` kullanırım."
        
- **INTERSECT:** İki sorgunun kesişim kümesini (ortak olanları) getirir.
    
- **EXCEPT (veya MINUS):** Birinci sorguda olup, ikinci sorguda olmayanları getirir (Fark kümesi).
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework ve LINQ:** Sen C# yazarken `db.Users.Where(u => u.Age > 18 && u.City == "Istanbul")` yazdığında, EF Core bunu arka planda SQL `AND` operatörüne çevirir. Bu çevirinin mantığını bilmek, karmaşık sorgularda (özellikle `OR` ve parantez kullanımında) hata yapmanı engeller.
    
2. **Arama Fonksiyonları:** Bir e-ticaret sitesinde "Ürün ara" kutusu yaparken `Contains("elma")` metodunu kullanırsın. Bu SQL'de `LIKE '%elma%'` operatörüne dönüşür. Bunun performans maliyetini bilmelisin (Index kullanımını etkiler).
    
3. **Raporlama:** Patronun "Geçen ay satış yapmayan kullanıcıları getir" dediğinde `EXCEPT` operatörü veya `IS NULL` mantığı hayat kurtarır.