
`INSERT`, veritabanındaki bir tabloya **yeni bir satır (kayıt)** eklemek için kullanılan DML komutudur. Tabloyu boş bir Excel sayfası gibi düşünürsen, `INSERT` komutu en alta yeni bir satır yazmaktır. İki şekilde kullanılır: Ya tüm sütunlara sırasıyla veri gönderirsin ya da sadece belirli sütunları seçip onlara veri gönderirsin.

**Neden Kullanılır?** Uygulamanda "Kayıt Ol" butonuna bastığında, "Sepete Ekle" dediğinde veya bir blog yazısı paylaştığında arka planda çalışan komut budur. Veritabanının büyümesini sağlayan ana komuttur.

---

### Deep Dive: Veri Eklemenin İncelikleri

Bir Junior Developer olarak `INSERT` yazmak kolaydır ama **performanslı** ve **güvenli** `INSERT` yazmak ustalık ister. Mülakatlarda özellikle "Bulk Insert" ve "Identity" konuları sorulur.

#### 1. Sütun Belirtme Kuralı (Best Practice)

İki tür yazım şekli vardır, mülakatta hangisini tercih ettiğin sorulur.

- **Riskli Yöntem (Kullanma):** `INSERT INTO Users VALUES ('Ahmet', 25)`
    
    - Burada sütun isimlerini yazmadık. SQL, sırasıyla ilk sütuna 'Ahmet', ikinciye 25 yazar.
        
    - **Tehlike:** Yarın öbür gün tablonun yapısı değişir ve araya "Soyad" sütunu eklenirse, senin kodun 'Ahmet'i isme, 25'i soyada yazmaya çalışır veya patlar. Buna "Schema Drift" kırılganlığı denir.
        
- **Güvenli Yöntem (Kullan):** `INSERT INTO Users (Name, Age) VALUES ('Ahmet', 25)`
    
    - Hangi verinin hangi sütuna gideceğini açıkça belirtirsin. Tabloya 100 tane yeni sütun eklense bile senin kodun çalışmaya devam eder.
        

#### 2. Identity (Otomatik Artan ID) ile Çalışmak

Tabloda `Id` sütunu genellikle `IDENTITY(1,1)` (Otomatik Artan) olarak ayarlanır.

- **Kural:** `INSERT` komutunda `Id` sütununa değer gönderemezsin! SQL bunu kendisi yönetir.
    
- **İstisna:** Eğer illaki kendi ID'mi vereceğim dersen `SET IDENTITY_INSERT TableName ON` komutunu kullanman gerekir (Junior seviyesinde pek önerilmez).
    

#### 3. Bulk Insert (Toplu Ekleme) - _Performans Sorusu_

Sana 1000 tane kullanıcıyı veritabanına ekle dediler.

- **Yavaş (Döngü ile):**
    
    SQL
    
    ```sql
    INSERT INTO Users (Name) VALUES ('Ahmet');
    INSERT INTO Users (Name) VALUES ('Mehmet');
    -- ... 1000 kere git gel yapar.
    ```
    
- **Hızlı (Bulk):**
    
    SQL
    
    ```sql
    INSERT INTO Users (Name) VALUES ('Ahmet'), ('Mehmet'), ('Ayşe'), ...;
    ```
    
    Tek bir komutla binlerce kayıt ekler. Sunucuya 1000 kere değil 1 kere gidersin.
    

#### 4. Eklenen ID'yi Geri Alma (`OUTPUT` / `SCOPE_IDENTITY`)

Bir kullanıcıyı kaydettin ve hemen ardından o kullanıcının ID'siyle ona profil resmi ekleyeceksin. ID'yi nasıl öğrenirsin?

- **SELECT MAX(ID):** Yanlıştır! Sen ekledikten milisaniye sonra başkası kayıt olursa onun ID'sini alırsın.
    
- **OUTPUT INSERTED.Id:** Doğru yöntemdir. Insert işlemiyle aynı anda oluşan ID'yi sana geri döner.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core `Add` vs `AddRange`:**
    
    - Tek kayıt için: `context.Users.Add(user);` -> `INSERT INTO...`
        
    - Çoklu kayıt için: `context.Users.AddRange(userList);` -> EF Core bunu otomatik olarak **Bulk Insert** komutuna çevirir. Eğer döngü (foreach) içinde tek tek `Add` yapıp `SaveChanges` dersen performansı öldürürsün. Bu farkı bilmek mülakatta hayat kurtarır.
        
2. **SQL Injection:** Eğer `INSERT` sorgusunu string birleştirerek (`"INSERT INTO... VALUES ('" + txtName.Text + "')"` gibi) yazarsan, hackerlar veritabanını ele geçirebilir. Parametreli sorgu veya ORM kullanmak zorunludur.
    
3. **Transaction Bütünlüğü:** Sipariş oluştururken hem `Orders` tablosuna hem de `OrderDetails` tablosuna `INSERT` yaparsın. İkincisi başarısız olursa birincisini de iptal etmelisin (Rollback). `INSERT` işlemleri genellikle bir Transaction bloğu içinde yönetilir.