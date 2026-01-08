Aggregate sorgular, veritabanındaki çok sayıda satırı alıp, üzerlerinde matematiksel işlemler yaparak **tek bir özet sonuca** dönüştüren fonksiyonlardır.

**Neden Kullanılır?** Bir e-ticaret siten var ve "Toplam kaç sipariş aldım?" veya "Bu ayın ciro ortalaması nedir?" sorularına cevap arıyorsun. Veritabanındaki milyonlarca satırı tek tek çekip Python veya C# tarafında döngüyle toplamak (Loop) hem çok yavaştır hem de RAM'i patlatır. Bu işi veritabanı motoruna yıkmak için Aggregate fonksiyonlarını kullanırız.

---

### Deep Dive: Fonksiyonların Gizli Tehlikeleri

Bir Junior Developer olarak `SUM` veya `COUNT` yazmak kolaydır. Ancak mülakatlarda seni eleyecekleri nokta **NULL (Boş)** değerlerin bu fonksiyonlarla nasıl etkileşime girdiğidir.

#### 1. COUNT() - _En Ünlü Mülakat Sorusu_

Soru: _`COUNT(*)` ile `COUNT(ColumnName)` arasındaki fark nedir?_

- **COUNT(*):** Tablodaki **tüm satırları** sayar. İçerik boş mu dolu mu bakmaz. Sadece "Kaç satır var?" sorusunun cevabıdır.
    
- **COUNT(ColumnName):** Sadece o sütundaki **NULL olmayan** (dolu) verileri sayar.
    
    - _Örnek:_ 100 kullanıcın var ama sadece 80 tanesinin e-postası kayıtlı (20'si NULL).
        
    - `COUNT(*)` -> 100 döner.
        
    - `COUNT(Email)` -> 80 döner.
        

#### 2. SUM() ve Veri Tipi Taşması (Overflow)

- **Risk:** `INT` veri tipinin sınırı yaklaşık 2 milyardır. Eğer satışların toplamı bu sayıyı geçerse SQL hata verir: _"Arithmetic overflow error converting expression to data type int."_
    
- **Çözüm:** Büyük toplamlar alırken sonucu daha büyük bir kaba (`BIGINT` veya `DECIMAL`) koymalısın.
    
    - `SELECT SUM(CAST(Fiyat AS BIGINT)) FROM Satislar`
        

#### 3. AVG() ve Tamsayı Bölmesi (Integer Division) Tuzağı

Bu hatayı yaparsan raporların yanlış çıkar.

- **Senaryo:** 1 ve 2 sayılarının ortalamasını alalım. Sonuç 1.5 olmalı değil mi?
    
- **SQL Davranışı:** Eğer sütun tipi `INT` ise, SQL sonucu tamsayıya yuvarlar. `AVG` sonucu **1** döner. (Küsüratı atar).
    
- **Çözüm:** Hassas ortalama için veriyi ondalıklı sayıya çevirmelisin.
    
    - `AVG(Fiyat * 1.0)` veya `AVG(CAST(Fiyat AS DECIMAL))`
        

#### 4. MIN() ve MAX() - Sadece Sayılar İçin Değil

Genelde sayısal düşünülür ama Tarih ve Metin için de çalışır.

- **Tarih:** `MIN(CreatedDate)` -> En eski kayıt tarihini verir.
    
- **Metin:** `MIN(Name)` -> Alfabetik olarak ilk ismi (Örn: "Ahmet") verir.
    

#### 5. Genel Kural: NULL Görmezden Gelinir

`COUNT(*)` hariç, tüm aggregate fonksiyonları (`SUM`, `AVG`, `MIN`, `MAX`) hesaplama yaparken **NULL değerleri yok sayar**.

- _Örnek:_ 3 satır var: `10`, `NULL`, `20`.
    
- `AVG` hesabı: (10 + 20) / 2 = 15 yapar. (NULL satırı hiç yokmuş gibi davranır, paydaya dahil etmez).
    

---

### Backend Developer İçin Neden Önemli?

1. **Server-Side vs Client-Side Evaluation:**
    
    - **Yanlış:** Tüm kullanıcıları veritabanından çekip (`SELECT * FROM Users`), kod tarafında `users.Count()` demek. (1 milyon veriyi ağ üzerinden taşırsın, sunucuyu kilitler).
        
    - **Doğru:** `SELECT COUNT(*) FROM Users` demek. (Ağ üzerinden sadece "1000000" sayısı gelir, milisaniye sürer).
        
    - _ORM:_ Entity Framework'te `context.Users.Count()` yazdığında, EF bunu otomatik olarak doğru SQL sorgusuna çevirir. Ama `context.Users.ToList().Count()` dersen felaket olur.
        
2. **Dashboard Performansı:** Admin panellerinde gördüğün o istatistik kutucukları (Widget), arkada bu aggregate sorgularını çalıştırır. Eğer bu sorgular yavaşsa (`SUM` yaparken Index kullanmıyorsan), admin paneli açılmaz.
    
3. **Veri Analizi:** Bir e-ticaret sitesinde "En pahalı ürün" (`MAX`) veya "Stoktaki toplam ürün değeri" (`SUM`) gibi bilgiler, iş kararları vermek için kritiktir. Sen backend developer olarak bu verileri sağlayacak API'leri yazarsın.