Veritabanındaki verilerin %70'inden fazlası metin (text) tabanlıdır. Ancak kullanıcılar veriyi her zaman senin istediğin gibi girmez. İsimleri küçük harfle yazarlar, telefon numaralarına boşluk koyarlar veya gereksiz karakterler eklerler. String Functions, bu dağınık veriyi düzenlemek ve analiz etmek için kullandığımız operatörlerdir.

Verdiğin 6 başlığı, bir Backend Developer gözüyle inceleyelim:

---

### 1. CONCAT (Birleştirme)

İki veya daha fazla metni yan yana getirip tek bir metin yapar. C#'taki `+` operatörü veya `string.Concat` gibidir.

- **Kullanım:** Genellikle "Ad" ve "Soyad" sütunlarını birleştirip "Tam Ad" yapmak için kullanılır.
    
- **SQL:** `SELECT CONCAT(FirstName, ' ', LastName) FROM Users`
    
- **Püf Nokta:** Bazı eski SQL sürümlerinde `+` ile birleştirme yapılırken, eğer verilerden biri `NULL` ise sonuç komple `NULL` olurdu. `CONCAT` fonksiyonu ise `NULL` veriyi boşluk (empty string) gibi algılar ve birleştirmeye devam eder. Bu yüzden daha güvenlidir.
    

### 2. LENGTH (Uzunluk) - _SQL Server Kullananlar Dikkat!_

Metnin kaç karakterden oluştuğunu döner.

- **Kullanım:** Validasyon için harikadır. "Şifresi 8 karakterden kısa olanları bul" veya "Telefon numarası 10 haneli olmayanları getir" gibi.
    
- **Kritik Fark:** Standart SQL'de adı `LENGTH` olsa da, **Microsoft SQL Server (T-SQL)** bu fonksiyonun adını **`LEN`** olarak kullanır. Eğer MSSQL çalışıyorsan `LEN(ColumnName)` yazmalısın.
    
- **Örnek:** `SELECT * FROM Users WHERE LEN(Password) < 8`
    

### 3. SUBSTRING (Parça Alma) - _En Büyük Tuzak_

Bir metnin içinden belirli bir parçayı kesip almak için kullanılır.

- **Kullanım:** "TC Kimlik numarasının son 2 hanesini al" veya "Adının sadece baş harfini al" gibi durumlarda kullanılır.
    
- **Syntax:** `SUBSTRING(Metin, BaşlangıçSırası, KaçKarakter)`
    
- **Yazılımcı Tuzağı (0 vs 1):** C#, Python, Java gibi dillerde index **0'dan başlar**. SQL'de ise index **1'den başlar!**
    
    - `SUBSTRING('Gemini', 1, 3)` -> **Gem** (C#'ta olsa 'emi' verirdi). Bu fark mülakatlarda ve kodlamada çok hata yaptırır.
        

### 4. REPLACE (Değiştirme)

Metin içindeki belirli bir karakteri veya kelimeyi bulup, başka bir şeyle değiştirir.

- **Kullanım:** Veri temizliği (Data Cleaning) için birebirdir.
    
- **Senaryo:** Kullanıcı telefon numarasını `(555) 123-4567` diye kaydetmiş. Sen bunu API'ye gönderirken sadece rakam olsun istiyorsun.
    
- **Örnek:** `REPLACE(REPLACE(Phone, '-', ''), ' ', '')` -> Tireleri ve boşlukları siler, `5551234567` kalır.
    

### 5. UPPER (Büyük Harf)

Tüm karakterleri BÜYÜK HARFE çevirir.

- **Kullanım:** Raporlamada başlıkları standartlaştırmak için kullanılır.
    

### 6. LOWER (Küçük Harf)

Tüm karakterleri küçük harfe çevirir.

- **Kullanım (Arama Motoru Mantığı):** Veritabanında arama yaparken (Case-Insensitive Search) en büyük kurtarıcıdır.
    
- **Senaryo:** Kullanıcı "iphone" arattı. Veritabanında "iPhone" veya "IPHONE" yazıyor olabilir.
    
- **Çözüm:** Hem aranan kelimeyi hem de veritabanındaki veriyi `LOWER` veya `UPPER` ile aynı formata getirip kıyaslarız.
    
    - `WHERE LOWER(ProductName) = 'iphone'` -> Böylece büyük/küçük harf fark etmeksizin eşleşme sağlanır.
        

---

### Backend Developer İçin Neden Önemli?

1. **EF Core Translation:** Sen C# tarafında `user.Name.ToLower() == "ahmet"` yazdığında, Entity Framework bunu arka planda otomatik olarak `LOWER(Name) = 'ahmet'` SQL koduna çevirir. Bu fonksiyonları bilmek, LINQ sorgularının arkada neye dönüştüğünü anlamanı sağlar.
    
2. **Veri Normalizasyonu:** Kullanıcı kayıt olurken e-posta adresini `AhMeT@GmAiL.CoM` diye yazsa bile, veritabanına kaydederken veya sorgularken `LOWER` kullanarak `ahmet@gmail.com` olarak işlem yapmak, ilerde oluşacak "Kullanıcı bulunamadı" hatalarını engeller.