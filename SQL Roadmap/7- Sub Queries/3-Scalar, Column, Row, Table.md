Genellikle SQL eğitimlerinde bu detay atlanır ama Subquery'leri (Alt Sorguları) anlamanın en iyi yolu, **"Geriye ne döndürüyor?"** sorusunu sormaktır. Verinin şekline göre (Tek hücre mi, liste mi, tablo mu?) kullanım yerin değişir.

Bu 4 başlığı "Subquery Çıktı Türleri" olarak derinlemesine inceleyelim:

---

### 1. Scalar (Tek Değer - Skaler)

Dönen sonuç **1 Satır ve 1 Sütun**dan oluşur. Yani tek bir hücredir.

- **Matematiksel Karşılığı:** Bir sayı, bir string veya bir tarih. (Örn: `500`, `"Ahmet"`, `'2024-01-01'`)
    
- **Kullanım Yeri:** Bir değişkene atanabildiği her yerde kullanılır. `SELECT` listesinde, `WHERE` bloğunda `=`, `>`, `<` gibi operatörlerle.
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Geriye sadece tek bir sayı (Ortalama) döner.
    SELECT * FROM Products
    WHERE Price > (SELECT AVG(Price) FROM Products);
    ```
    
- **Kritik Hata (Mülakat Sorusu):** Eğer Scalar beklenen bir yere (örneğin `=`), birden fazla satır döndüren bir sorgu yazarsan şu hatayı alırsın: _"Subquery returned more than 1 value. This is not permitted..."_
    

### 2. Column (Sütun - Liste)

Dönen sonuç **Tek Sütun ve Çok Satır**dan oluşur. Yani bir listedir.

- **Yazılım Karşılığı:** `List<int>` veya `Array[string]`.
    
- **Kullanım Yeri:** Tek bir değer olmadığı için `=`, `!=` gibi operatörlerle **kullanılamaz**. Bunun yerine `IN`, `NOT IN`, `ANY`, `ALL` gibi küme operatörleriyle kullanılır.
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Geriye bir ID listesi döner (Örn: 1, 5, 9).
    SELECT Name FROM Users
    WHERE Id IN (SELECT UserId FROM Orders WHERE Total > 1000);
    ```
    

### 3. Row (Satır - Kayıt)

Dönen sonuç **Çok Sütun ama Tek Satır**dan oluşur. Yani tek bir kayıttır.

- **Yazılım Karşılığı:** Tek bir `Object` veya `Class` instance'ı (Örn: `User` nesnesi).
    
- **Kullanım Yeri:** Genellikle karşılaştırmalarda veya `SELECT` içinde birden fazla alanı tek seferde çekmek istediğinde kullanılır. (Bazı veritabanlarında `WHERE (Col1, Col2) = (SELECT Col1, Col2 ...)` şeklinde tuple karşılaştırması yapılabilir).
    
- **Örnek:** En pahalı ürünün adını ve fiyatını aynı anda getirmek.
    

### 4. Table (Tablo - Matris)

Dönen sonuç **Çok Sütun ve Çok Satır**dan oluşur. Yani tam teşekküllü sanal bir tablodur.

- **Yazılım Karşılığı:** `DataTable` veya `List<Object>`.
    
- **Kullanım Yeri:** Bir tablo olduğu için genellikle `FROM` veya `JOIN` bloklarında kullanılır. Buna **Derived Table** (Türetilmiş Tablo) denir.
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- İçerideki sorgu sanal bir tablo oluşturur, dışarıdaki sorgu onu kullanır.
    SELECT * FROM
    (SELECT Name, Price, CategoryId FROM Products WHERE Price > 100) AS PahaliUrunler
    WHERE CategoryId = 5;
    ```
    

---

### Backend Developer İçin Neden Önemli?

1. **ADO.NET Metotları:** .NET'in en temel veritabanı erişim kütüphanesi olan ADO.NET'te bu tipler için özel metotlar vardır. Mülakatta sorarlarsa +100 puan:
    
    - **Scalar Subquery için:** `command.ExecuteScalar()` (Sadece ilk satırın ilk hücresini döndürür, çok hızlıdır).
        
    - **Table/Row Subquery için:** `command.ExecuteReader()` (Veriyi tablo olarak okur).
        
    - **Column Subquery için:** Veriyi Reader ile okuyip bir `List<>` içine atarsın.
        
2. **Entity Framework Core `SingleOrDefault` vs `ToList`:**
    
    - Eğer bir sorgunun **Scalar** veya **Row** (Tek kayıt) döneceğinden eminsen `.First()` veya `.Single()` kullanırsın.
        
    - Eğer **Column** veya **Table** (Liste) dönecekse `.ToList()` kullanırsın.
        
    - Yanlış seçim (örneğin çok satır dönen yere `Single` yazmak), çalışma zamanında (Runtime) uygulamanın patlamasına ("Sequence contains more than one element") neden olur.