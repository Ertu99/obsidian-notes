`FROM` komutu, `SELECT` sorgusunun hedeflediği verilerin **hangi tablo, view veya alt sorgudan** çekileceğini belirttiğimiz kısımdır.

**Neden Kullanılır?** Veritabanına "Bana 'Ad' ve 'Soyad' sütunlarını getir" (`SELECT Name, Surname`) dersen, veritabanı sana "Hangi tablodan?" diye sorar. `FROM` bu sorunun cevabıdır. Sorgunun veri kaynağını tanımlar.

---

### Deep Dive: FROM Bloğunun Görünmeyen Gücü

Bir Junior Developer olarak `FROM`'u sadece tablo ismi yazılan yer olarak görebilirsin ama SQL motoru (Engine) için sorgunun **başlangıç noktasıdır**.

#### 1. SQL Sorgularının Çalışma Sırası (Order of Execution) - _En Önemli Kısım_

Mülakatta "SQL sorgusu nasıl çalışır?" dendiğinde C# gibi yukarıdan aşağıya (SELECT'ten başlayarak) çalıştığını sanıyorsan yanılırsın.

SQL Motoru sorguyu şu sırayla işler:

1. **FROM** (Önce tabloyu bulur ve hafızaya alır).
    
2. **WHERE** (Verileri filtreler).
    
3. **GROUP BY** (Gruplar).
    
4. **HAVING** (Grupları filtreler).
    
5. **SELECT** (Hangi sütunların gösterileceğini seçer).
    
6. **ORDER BY** (Sıralar).
    

**Mülakat Sorusu:** _"Neden `SELECT Isim AS Ad` diyip, `WHERE Ad = 'Ali'` diyemiyorum?"_ **Cevap:** Çünkü `WHERE` bloğu çalıştığı anda henüz `SELECT` bloğu çalışmamıştır. Veritabanı "Ad" diye bir takma isim (Alias) olduğunu henüz bilmiyordur. Bu yüzden `FROM` ve işlem sırası mantığını bilmek hayati önem taşır.

#### 2. Alias (Takma İsim) Kullanımı

Tablo isimleri bazen çok uzun olabilir (`ECommerce_System_User_Table` gibi) veya aynı tabloyu kendisiyle birleştirmen gerekebilir.

- **Kullanım:** `FROM Users AS u` veya kısaca `FROM Users u`.
    
- Artık sorgunun geri kalanında `Users.Name` yerine `u.Name` yazabilirsin. Kod okunabilirliği için standarttır.
    

#### 3. Çoklu Tablo ve "Virgül" Tuzağı (Cartesian Product)

Eski SQL standartlarında (ANSI-89), tabloları birleştirmek için `JOIN` kelimesi yerine `FROM` içinde virgül kullanılırdı.

- **Sorgu:** `SELECT * FROM Users, Orders`
    
- **Tehlike:** Eğer `WHERE` koşulu ile bu iki tabloyu bağlamazsan, SQL veritabanındaki **her kullanıcıyı her siparişle** eşleştirir.
    
    - 100 Kullanıcı ve 100 Sipariş varsa, sonuç **10.000** satır döner.
        
    - Buna **Kartezyen Çarpım (Cartesian Product / Cross Join)** denir.
        
    - **Öneri:** Modern SQL'de virgül ile birleştirme **yapma**. Her zaman açıkça `INNER JOIN`, `LEFT JOIN` kullan.
        

#### 4. Subquery (Alt Sorgu) Olarak FROM

`FROM` sadece gerçek bir tabloyu değil, o an ürettiğin sanal bir tabloyu da kaynak olarak alabilir. Buna **Derived Table** denir.

SQL

```
SELECT * FROM (SELECT Name, Age FROM Users WHERE Age > 18) AS Yetiskinler
WHERE Name LIKE 'A%';
```

Burada `FROM`, veritabanındaki bir tabloya değil, parantez içindeki sonucun kendisine bakmaktadır.

---

### Backend Developer İçin Neden Önemli?

1. **LINQ Sorguları:** .NET'te LINQ yazarken sözdizimi SQL'in çalışma mantığına göre tersten başlar: `from u in context.Users select u.Name` C# (LINQ), SQL'in doğal işlem sırasını (önce FROM) taklit eder. Bunu anlamak LINQ sorgularını daha rahat yazmanı sağlar.
    
2. **Performans (NOLOCK):** Yüksek trafikli sistemlerde `FROM Users WITH (NOLOCK)` gibi ifadeler görebilirsin. Bu, "Veri okunurken tabloyu kilitleme, kirli veri (dirty read) gelse de olur, yeter ki hızlı olsun" demektir. Backend developer olarak raporlama ekranlarında bu ipuçlarını `FROM` bloğuna eklersin.