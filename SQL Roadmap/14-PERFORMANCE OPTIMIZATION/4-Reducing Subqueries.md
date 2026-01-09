

SQL performans dünyasında "Subquery" (Alt Sorgu) iki yüzlü bir madalyondur. Bazen hayat kurtarır, bazen performansı yerle bir eder.

Özellikle **Correlated Subquery (İlişkili Alt Sorgu)** dediğimiz tür, veritabanı motorunun kâbusudur. Mantığı şudur: "Dışarıdaki sorgunun her bir satırı için, içerideki sorguyu tekrar çalıştır." Eğer dışarıda 1 milyon satır varsa, içerdeki sorgu 1 milyon kere çalışır. Buna "N+1 Problemi"nin SQL versiyonu diyebiliriz.

Bu yükten kurtulmak için kullanılan 3 ana stratejiyi inceleyelim:

---

### 1. Subquery yerine JOIN Kullanmak

En büyük performans artışı buradan gelir. Döngüsel işlemi (Loop), küme bazlı (Set-based) işleme çevirirsin.

- **Kötü (Correlated Subquery):** Her kullanıcı için sipariş tablosuna tek tek gidip sayım yapar.
    
    SQL
    
    ```sql
    SELECT
        u.Name,
        (SELECT COUNT(*) FROM Orders o WHERE o.UserId = u.Id) AS OrderCount
    FROM Users u;
    ```
    
- **İyi (LEFT JOIN + GROUP BY):** İki tabloyu tek seferde birleştirir ve gruplar. Çok daha hızlıdır.
    
    SQL
    
    ```sql
    SELECT
        u.Name,
        COUNT(o.Id) AS OrderCount
    FROM Users u
    LEFT JOIN Orders o ON u.Id = o.UserId
    GROUP BY u.Name;
    ```
    

### 2. CTE (Common Table Expressions) ile Okunabilirlik

Prompt'unda bahsettiğin CTE'ler (`WITH` bloğu), özellikle aynı alt sorguyu birden fazla yerde kullanıyorsan harikadır.

- **Neden Kullanılır?** İç içe geçmiş parantezler (`SELECT ... FROM (SELECT ... FROM ...)`) kodu okunmaz hale getirir ("Spaghetti Code"). CTE, kodu modüler parçalara böler.
    
- **Performans Notu:** SQL Server'da CTE'ler genelde performans artışı sağlamaz (Sadece okumayı kolaylaştırır), ancak PostgreSQL gibi bazı veritabanlarında `MATERIALIZED` özelliği ile sonucu hafızada tutup performansı artırabilir.
    

### 3. EXISTS vs. IN (Kritik Ayrım)

Bir kaydın varlığını kontrol ederken `IN` kullanmak yerine `EXISTS` kullanmak genellikle daha performanslıdır.

- **IN:** Parantez içindeki tüm listeyi çeker, hafızaya alır ve karşılaştırır.
    
- **EXISTS:** Aradığı şartı sağlayan **ilk kaydı** bulduğu an durur (Short-Circuit). "Var mı? Var. Tamam gerisine bakmaya gerek yok" der.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core Çıktısı:** LINQ ile yazdığın karmaşık sorgular (`.Select(x => new { Count = x.Orders.Count() })`), arka planda veritabanına bir **Correlated Subquery** olarak gönderilebilir.
    
    - Profiler ile baktığında `OUTER APPLY` veya alt alta `SELECT` görüyorsan, LINQ sorgunu `GroupJoin` veya düzgün bir `Include` yapısına çevirmen gerekebilir.
        
2. **Temp Table Stratejisi:** Çok karmaşık bir rapor hazırlıyorsun. Aynı hesaplamayı (örneğin: `GecenYilCiro`) raporun 5 farklı yerinde kullanıyorsun.
    
    - Bunu her seferinde Subquery ile hesaplatmak yerine, önce bir `#TempTable` içine hesaplayıp atarsın. Sonraki adımlarda bu hazır tabloyu kullanırsın. İşlemciye nefes aldırırsın.