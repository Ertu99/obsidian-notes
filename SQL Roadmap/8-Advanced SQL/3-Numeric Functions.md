Matematiksel fonksiyonlar, veritabanındaki sayısal veriler üzerinde aritmetik işlemler yapmamızı sağlar. "Hesaplamayı C# tarafında yaparım" diyebilirsin ama milyonlarca satırlık bir finans raporu hazırlarken veya sayfalama (pagination) yaparken bu yükü SQL motoruna yıkmak en doğrusudur.

Verdiğin 5 fonksiyonu, kullanım alanlarına göre gruplayarak inceleyelim:

---

### 1. Yuvarlama Ekibi: FLOOR, CEILING ve ROUND

Bu üçlü sık sık karıştırılır ama aralarındaki fark çok nettir.

- **FLOOR (Zemin / Taban):** Sayiyi her zaman **bir alt tamsayıya** indirir. Yerçekimi gibidir, sayıyı aşağı çeker.
    
    - `FLOOR(5.9)` -> **5** (Virgülden sonrasına bakmaz, acımasızca atar).
        
    - `FLOOR(-5.1)` -> **-6** (Dikkat! Negatif sayılarda daha küçüğe gider).
        
- **CEILING (Tavan):** Sayiyi her zaman **bir üst tamsayıya** çıkarır.
    
    - `CEILING(5.1)` -> **6** (Çok az bir fazlalık olsa bile yukarı tamamlar).
        
    - **Gerçek Hayat Örneği (Pagination):** E-ticaret sitende 101 ürün var ve her sayfada 10 ürün gösteriyorsun. Kaç sayfa lazım?
        
        - 101 / 10 = 10.1 sayfa.
            
        - `FLOOR` kullanırsan 10 sayfa der, son ürün kaybolur.
            
        - `CEILING` kullanırsan 11 sayfa der, son ürün 11. sayfada görünür. Bu yüzden sayfalama mantığında her zaman `CEILING` kullanılır.
            
- **ROUND (Matematiksel Yuvarlama):** İlkokulda öğrendiğimiz "buçuklu sayı" kuralıdır. En yakın tamsayıya (veya belirtilen ondalığa) yuvarlar.
    
    - `ROUND(5.4, 0)` -> **5** (5.5'ten küçük olduğu için aşağı).
        
    - `ROUND(5.5, 0)` -> **6** (5.5 ve üzeri olduğu için yukarı).
        
    - **Hassasiyet:** `ROUND` fonksiyonu ikinci bir parametre alarak virgülden sonra kaç basamak kalacağını belirleyebilir. Fiyat hesaplamalarında (KDV vb.) kuruş hassasiyeti için `ROUND(Fiyat, 2)` kullanılır.
        

---

### 2. Mutlak Değer: ABS (Absolute)

- **Ne Yapar:** Negatif sayıları pozitife çevirir. Pozitiflere dokunmaz.
    
- **Mantık:** Sayının 0 noktasına olan uzaklığıdır. Uzaklık eksi olamaz.
    
- **Kullanım:** İki sayı arasındaki farkı bulurken sonucun eksi çıkmasını istemiyorsan kullanırsın.
    
    - Senaryo: A kişisinin puanı 100, B kişisinin puanı 150. Aradaki farkı bul. `100 - 150 = -50`. Ama biz "Fark 50 puan" demek istiyoruz.
        
    - `ABS(100 - 150)` -> **50**.
        

### 3. Kalan Bulucu: MOD (Modulo)

- **Ne Yapar:** Bir bölme işleminden kalan sayıyı verir.
    
- **Syntax:** Bazı veritabanlarında `MOD(10, 3)`, bazılarında (SQL Server gibi) `%` operatörü (`10 % 3`) kullanılır.
    
    - 10'un 3'e bölümünden kalan 1'dir. Sonuç: **1**.
        
- **Kullanım Alanları:**
    
    - **Tek/Çift Kontrolü:** Sayının 2'ye bölümünden kalan 0 ise Çift, 1 ise Tek sayıdır.
        
    - **Dağıtım (Sharding):** Milyonlarca kullanıcıyı 3 farklı sunucuya eşit dağıtmak istiyorsun.
        
        - `SunucuNo = UserId % 3`
            
        - Sonuç her zaman 0, 1 veya 2 çıkar. Kullanıcıları bu şekilde sunuculara paylaştırırsın.
            

---

### Backend Developer İçin Neden Önemli?

1. **Pagination (Sayfalama):** Yukarıda bahsettiğim gibi, bir API yazarken "Toplam Sayfa Sayısı"nı hesaplamak zorundasın. Bunu yaparken `Math.Ceiling` (C#) veya SQL `CEILING` fonksiyonu olmazsa olmazdır.
    
2. **Finansal İşlemler:** Bankacılık veya e-ticaret uygulamasında "Kuruş" hesapları çok kritiktir. 1 kuruşluk farklar bile maliye ile sorun yaratır. Veritabanına kaydederken veya veriyi çekerken `ROUND(Tutar, 2)` kullanmak standarttır.
    
3. **İş Mantığı (Business Logic):** `MOD` fonksiyonu, belirli periyotlarda işlem yapmak için kullanılır. Örneğin "Her 100. müşteriye hediye ver" demek için `WHERE MusteriSiraNo % 100 = 0` sorgusu yazılır