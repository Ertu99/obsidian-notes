Subquery (veya Nested Query), bir SQL sorgusunun içine gömülmüş başka bir SQL sorgusudur. Türkçesiyle "Sorgu İçinde Sorgu" veya "Yavru Sorgu" diyebiliriz.

**Neden Kullanılır?** Matematikteki parantez içi işlemler gibidir. Ana sorgunun (Outer Query) çalışabilmesi için önce içerideki küçük sorgunun (Inner Query) bir cevap üretmesi gerekir. Karmaşık problemleri parçalara ayırmak veya dinamik filtrelemeler yapmak için kullanılır.

---

### Deep Dive: "Inception" Mantığı ve Teknik Detaylar

Bir Junior Developer olarak Subquery yazmak, JOIN yazmaktan daha kolay gelebilir çünkü mantığı "adım adım" ilerler. Ancak mülakatlarda ve performansta dikkat etmen gereken ince çizgiler vardır.

#### 1. Çalışma Mantığı ve Sırası

Genel kural şudur: **Önce içteki sorgu çalışır.**

- **Senaryo:** "Fiyatı, ortalama ürün fiyatından yüksek olan ürünleri getir."
    
- **Mantık:**
    
    1. Önce ortalamayı bulmam lazım. (İç Sorgu: `SELECT AVG(Price) FROM Products` -> Sonuç: 500 TL).
        
    2. Şimdi 500 TL'den pahalıları bulabilirim. (Dış Sorgu: `SELECT * FROM Products WHERE Price > 500`).
        
- **SQL:**
    
    SQL
    
    ```sql
    SELECT * FROM Products
    WHERE Price > (SELECT AVG(Price) FROM Products);
    ```
    
    _Not:_ `WHERE` bloğunda doğrudan `AVG()` kullanamayacağımız için (Aggregate fonksiyon kuralı), Subquery burada hayat kurtarıcıdır.
    

#### 2. Kullanım Yerleri (Konumuna Göre İsimleri)

Subquery'ler sorgunun neresinde durduğuna göre farklı davranır:

- **WHERE Bloğunda (Filtreleme):** En yaygın kullanımdır. Yukarıdaki örnekteki gibi bir değer döndürür ve filtre olarak kullanılır.
    
- **FROM Bloğunda (Derived Table):** Sanki gerçek bir tabloymuş gibi davranır.
    
    - `SELECT * FROM (SELECT Name, Age FROM Users) AS GeciciTablo WHERE Age > 18`
        
- **SELECT Bloğunda (Hesaplanmış Sütun):** Her satır için tek tek çalışır.
    
    - `SELECT Name, (SELECT COUNT(*) FROM Orders WHERE UserId = Users.Id) AS SiparisSayisi FROM Users`
        

#### 3. Correlated Subquery (İlişkili Alt Sorgu) - _Performans Katili_

Mülakatta sorarlar: _"Correlated Subquery nedir ve neden tehlikelidir?"_

- **Normal Subquery:** İçerideki sorgu 1 kere çalışır, sonucu dışarı verir. (Hızlıdır).
    
- **Correlated Subquery:** İçerideki sorgu, dışarıdaki tablonun her bir satırı için **tekrar tekrar** çalışır.
    
    - Eğer 1 milyon ürünün varsa ve `SELECT` bloğuna ilişkili bir subquery yazdıysan, içerdeki sorgu 1 milyon kere çalışır. Sunucuyu kilitler.
        

#### 4. Subquery vs JOIN - _Ezeli Rekabet_

Çoğu durumda Subquery ile yaptığın işi JOIN ile de yapabilirsin.

- **Hangisi Hızlı?** Genellikle **JOIN** daha hızlıdır (Veritabanı motorları JOIN optimizasyonunda daha iyidir).
    
- **Ne Zaman Subquery?** JOIN yazmak sorguyu çok karmaşıklaştırıyorsa veya `AVG` gibi aggregate değerlerle kıyaslama yapacaksan Subquery daha okunaklıdır.
    

---

### Backend Developer İçin Neden Önemli?

1. **LINQ Çevirileri:** C# tarafında yazdığın bazı LINQ sorguları, EF Core tarafından otomatik olarak Subquery'e dönüştürülür.
    
    - `context.Products.Where(p => p.Price > context.Products.Average(x => x.Price))`
        
    - Bu kodu yazdığında arka planda oluşan SQL, yukarıdaki Subquery örneğinin aynısıdır.
        
2. **Okunabilirlik (Readability):** Karmaşık raporlama sorgularında, 5 tane tabloyu JOINlemek yerine, veriyi önce bir Subquery (veya CTE) ile hazırlayıp sonra ana sorguda kullanmak kodu daha anlaşılır kılar. "Clean Code" prensibi SQL için de geçerlidir.
    
3. **Toplu Güncellemeler:** Bazen güncelleme yaparken veriyi başka tablodan alman gerekir. `UPDATE Products SET Price = (SELECT Price FROM OldProducts WHERE Id = Products.Id)` Burada Subquery zorunludur.
    

Subqueries konusu, özellikle "Önce iç çalışır" mantığı ve "Correlated" performans riskiyle birlikte tamamlandı.