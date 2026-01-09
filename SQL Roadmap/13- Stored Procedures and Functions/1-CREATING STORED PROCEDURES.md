Bir Backend Developer olarak C#'ta nasıl metod yazıyorsan (`public void SaveUser(...)`), SQL'de de Stored Procedure (SP) yazmak aynı mantıktır. Kodun veritabanı tarafında yaşar, sen sadece ismini çağırırsın.

**Temel Yapı:** Bir SP üç ana bölümden oluşur:

1. **Header (Başlık):** Adı ve parametreleri (`@Ad`, `@Soyad`).
    
2. **Execution (Gövde):** `AS` kelimesinden sonra gelen ve yapılacak işi anlatan SQL kodları.
    
3. **Return (Dönüş):** İsteğe bağlı olarak geriye veri veya durum kodu döndürme.
    

---

### Adım Adım Oluşturma

Seninle gerçekçi bir senaryo yapalım. Sisteme yeni kayıt olan bir kullanıcıyı ekleyen ve eklediği kullanıcının ID'sini geriye döndüren bir SP yazalım.

SQL

```sql
-- 1. İsimlendirme: Genelde 'sp_' veya 'usp_' (User Stored Proc) ön eki kullanılır.
CREATE PROCEDURE sp_RegisterUser
    -- 2. Parametreler: Dışarıdan gelecek veriler '@' ile başlar.
    @UserName NVARCHAR(50),
    @Email NVARCHAR(100),
    @PasswordHash NVARCHAR(256)
AS
BEGIN
    -- 3. Performans Ayarı (Önemli!)
    -- "1 row affected" mesajını kapatır. Ağ trafiğini (Network I/O) azaltır.
    SET NOCOUNT ON;

    -- 4. İşlem (Business Logic)
    INSERT INTO Users (Name, Email, Password)
    VALUES (@UserName, @Email, @PasswordHash);

    -- 5. Dönüş (Geriye yeni oluşan ID'yi verelim)
    SELECT SCOPE_IDENTITY() AS NewUserId;
END
```

---

### Backend Developer İçin Kritik Noktalar

#### 1. Parametreler (Input vs Output)

- **Input Parameters:** Yukarıdaki örnekteki gibi dışarıdan içeri veri sokar.
    
- **Output Parameters:** C#'taki `out` keyword'ü gibidir. Prosedür bitince dışarı veri fırlatır.
    
    - _Kullanım:_ `CREATE PROCEDURE sp_GetCount @Total INT OUTPUT ...`
        

#### 2. `SET NOCOUNT ON` (Gizli Kahraman)

Mülakatlarda "Prosedür yazarken performansı artırmak için ilk satıra ne yazarsın?" diye sorarlarsa cevabın bu olsun.

- **Neden?** SQL her işlemde (Insert/Update) geriye "1 satır etkilendi" mesajı döner. Binlerce işlem yapan bir döngüde bu mesajlar gereksiz ağ trafiği yaratır. Bunu kapatmak Backend performansını artırır.
    

#### 3. Değişken Tanımlama (`DECLARE`)

Prosedür içinde geçici hesaplamalar yapmak için değişken kullanabilirsin.

SQL

```sql
DECLARE @Bugun DATETIME = GETDATE(); -- Değişken oluştur
-- İşlemler...
```

#### 4. Transaction Kullanımı

SP içinde birden fazla tabloya yazıyorsan (örn: Hem `Users` hem `UserRoles`), veri bütünlüğü için `BEGIN TRANSACTION ... COMMIT` bloğunu SP'nin içine gömmek en profesyonel yöntemdir. Böylece Backend tarafında transaction yönetmekle uğraşmazsın.

---

### C# Tarafında Nasıl Görünür?

Sen bu prosedürü oluşturduktan sonra, C# tarafında (ADO.NET veya Dapper kullanarak) şöyle çağırırsın:

C#

```csharp
// SQL kodu yazmak yok, sadece prosedür adı var.
connection.Execute("sp_RegisterUser", new { UserName = "Ahmet", ... }, commandType: CommandType.StoredProcedure);
```

Bu sayede SQL tablosunun adı değişse bile C# kodun bozulmaz, sadece prosedürü güncellersin


### **EXECUTING STORED PROCEDURES (Prosedürleri Çalıştırma)**

Prosedürü yazdık ve kaydettik (`CREATE`). Şimdi sıra geldi o kodu çalıştırmaya (`EXECUTE`). Bu işlem, C#'ta yazdığın bir metodu çağırmakla (`CalculateTax();`) tamamen aynı mantıktır.

SQL'de prosedürleri çalıştırmak için **`EXEC`** veya **`EXECUTE`** komutu kullanılır.

---

### 1. Parametresiz Çalıştırma

Eğer prosedür dışarıdan veri istemiyorsa, sadece adını çağırmak yeterlidir.

SQL

```sql
EXEC sp_TumKullanicilariGetir;
```

### 2. Parametreli Çalıştırma (Kritik Ayrım)

Prosedüre değer gönderirken iki yöntem vardır. Backend developer olarak **İkinci Yöntemi** (Named Parameters) tercih etmen hayatını kurtarır.

#### A. Sıralı Gönderim (Positional - Riskli)

Değerleri, prosedürü oluştururken tanımladığın sıraya göre virgüle ayırarak gönderirsin.

SQL

```sql
-- sp_RegisterUser (@Name, @Email, @Password)
EXEC sp_RegisterUser 'Ahmet', 'ahmet@mail.com', '123456';
```

- **Risk:** Eğer yarın birisi prosedürdeki parametrelerin yerini değiştirirse (önce Email, sonra Name yaparsa), senin kodun Ahmet'i email adresi, mail adresini isim olarak kaydeder. Veri bozulur.
    

#### B. İsimli Gönderim (Named Parameters - Güvenli)

Hangi değerin hangi parametreye gideceğini açıkça belirtirsin. Sıra karışsa bile SQL bunu anlar.

SQL

```sql
EXEC sp_RegisterUser
    @Email = 'ahmet@mail.com',
    @Password = '123456',
    @Name = 'Ahmet'; -- Sıra değişse bile doğru çalışır
```

---

### 3. OUTPUT Parametre Yakalama (İleri Seviye)

Bazen prosedür sana sadece bir tablo değil, özel bir değer (örn: Yeni oluşan ID veya İşlem Sonucu) döndürür. Bunu yakalamak için dışarıda bir "Kova" (Değişken) hazırlaman gerekir.

SQL

```sql
-- 1. Kovayı hazırla
DECLARE @YeniId INT;

-- 2. Prosedürü çalıştır ve çıktıyı kovaya doldur (OUTPUT komutuyla)
EXEC sp_RegisterUser
    @UserName = 'Mehmet',
    @Email = 'mehmet@test.com',
    @NewUserId = @YeniId OUTPUT; -- Dönen veriyi @YeniId değişkenine yaz

-- 3. Sonucu gör
SELECT @YeniId AS 'Kaydedilen ID';
```

---

### Backend Developer İçin Neden Önemli?

1. **Dapper ve EF Core Kullanımı:** C# tarafında kod yazarken `CommandType` belirtmek performansı artırır.
    
    C#
    
    ```csharp
    // Dapper Örneği
    var p = new DynamicParameters();
    p.Add("@Name", "Ahmet");
    p.Add("@Email", "ahmet@mail.com");
    
    // CommandType.StoredProcedure demek, SQL'e "Bu bir düz yazı değil, derlenmiş kod" demektir.
    connection.Execute("sp_RegisterUser", p, commandType: CommandType.StoredProcedure);
    ```
    
2. **Debug (Hata Ayıklama):** Backend kodun hata veriyorsa, önce parametreleri alıp SQL Management Studio'da `EXEC` ile manuel olarak çalıştırırsın.
    
    - Eğer SQL'de çalışıyor ama C#'ta çalışmıyorsa sorun kodundadır.
        
    - SQL'de de hata veriyorsa sorun veritabanı mantığındadır. Sorunu izole etmeni sağlar.
        

Stored Procedure defterini kapattık. Şimdi bunların kardeşi olan ama kuralları çok daha katı olan