SQL'in en zeki olduğu kısma geldik. Şimdiye kadar veriyi hep olduğu gibi çektik veya matematiksel işlem yaptık. "Conditional" fonksiyonlar ise SQL'e **karar verme yeteneği** (Logic) kazandırır. Kod tarafında (C# veya Python) yazdığın `if-else` bloklarını veritabanı içinde yazmanı sağlar.

Verdiğin 3 hayati fonksiyonu inceleyelim:

---

### 1. CASE (SQL'in If-Else'i)

SQL'deki en güçlü koşullu ifadedir. Bir sütundaki veriye bakıp, duruma göre farklı bir çıktı üretmeni sağlar.

- **Mantık:** "Eğer durum buysa bunu yap, şuysa şunu yap, hiçbiri değilse bunu yap."
    
- **Yazılım Karşılığı:** C#'taki `switch-case` veya `if-else if-else` blokları.
    
- **Örnek Senaryo:** Sınav notlarına göre harf notu vermek.
    
    SQL
    
    ```sql
    SELECT StudentName, Score,
        CASE
            WHEN Score >= 90 THEN 'A'
            WHEN Score >= 70 THEN 'B'
            WHEN Score >= 50 THEN 'C'
            ELSE 'F' -- Hiçbiri tutmazsa (Else opsiyoneldir)
        END AS Grade
    FROM Students
    ```
    
- **Kritik Özellik:** `CASE` ifadesi sadece `SELECT` listesinde değil, `ORDER BY` içinde de kullanılabilir. (Örn: "Önce Aktif üyeleri, sonra Pasifleri sırala" gibi özel sıralamalarda).
    

### 2. NULLIF (Eşitse Yok Et)

İki değer birbirine **eşitse NULL döner**, eşit değilse **ilk değeri döner**.

- **Mantık:** `if (param1 == param2) return NULL; else return param1;`
    
- **Neden Kullanılır? (Division by Zero):** Yazılım dünyasında bir sayıyı 0'a bölersen program çöker (DivideByZeroException).
    
- **Mülakat Sorusu:** _"SQL'de ortalama hesaplarken paydanın 0 olma ihtimali var, hatayı nasıl engellersin?"_
    
- **Çözüm:**
    
    SQL
    
    ```sql
    -- Eğer Count 0 ise, NULLIF(Count, 0) işlemi NULL döner.
    -- Sayı / NULL işleminin sonucu SQL'de hata değil, NULL'dır. Program kurtulur.
    SELECT TotalPrice / NULLIF(ProductCount, 0) FROM Orders
    ```
    

### 3. COALESCE (İlk Doluyu Bul)

İçine verilen parametre listesindeki **ilk NULL OLMAYAN** (dolu) değeri döndürür.

- **Mantık:** "Yedek planı devreye sok."
    
- **Yazılım Karşılığı:** C#'taki `??` operatörü (Null-Coalescing Operator).
    
- **Örnek Senaryo:** Kullanıcının Cep Telefonu varsa onu getir, yoksa Ev Telefonunu getir, o da yoksa "Numara Yok" yaz.
    
    SQL
    
    ```sql
    SELECT Name,
           COALESCE(MobilePhone, HomePhone, WorkPhone, 'Numara Yok') AS ContactNumber
    FROM Users
    ```
    
- **Farkı Nedir? (ISNULL vs COALESCE):** SQL Server'da `ISNULL` fonksiyonu sadece 2 parametre alır. `COALESCE` ise sınırsız sayıda parametre alabilir ve ANSI SQL standardıdır (Her veritabanında çalışır).
    

---

### Backend Developer İçin Neden Önemli?

1. **API Response Formatlama:** Frontend tarafı senden veriyi "1" veya "0" olarak değil, "Aktif" veya "Pasif" olarak isteyebilir. Bunu C# kodunda `foreach` ile dönüp değiştirmek yerine, SQL'de `CASE WHEN Status = 1 THEN 'Aktif' ELSE 'Pasif' END` yazarak hazır çekmek performansı artırır.
    
2. **Raporlama ve Dashboardlar:** Yöneticiler raporda "NULL" görmek istemezler, "0" veya "-" görmek isterler. `SELECT COALESCE(Sum(Sales), 0)` diyerek, hiç satış olmayan günlerde NULL yerine 0 gelmesini sağlarsın. Bu sayede grafik kütüphaneleri patlamaz.
    
3. **Güvenli Matematik:** Finansal hesaplamalar yaparken `NULLIF` kullanmak, veritabanındaki hatalı (0 girilmiş) veriler yüzünden tüm raporun hata verip durmasını engeller. "Defensive Coding" (Savunmacı Kodlama) veritabanında başlar.