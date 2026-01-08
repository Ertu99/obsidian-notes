`CREATE TABLE`, veritabanı içinde verilerin saklanacağı kutuları (tabloları) ve bu kutuların kurallarını sıfırdan inşa eden temel DDL komutudur.

**Neden Kullanılır?** Veritabanını boş bir arsa gibi düşünürsen, `CREATE TABLE` o arsaya bina dikmektir. Verinin hangi formatta (sayı, yazı, tarih) saklanacağını, hangi verinin zorunlu olduğunu ve veriler arasındaki ilişkileri bu komutla en başta belirleriz. Bu iskelet kurulmadan veri girişi (INSERT) yapılamaz.

---

### Deep Dive: Bir Tablonun Anatomisi

Bir .NET Junior Developer olarak kod tarafında (C#) bir "Class" oluşturduğunda, bu aslında veritabanında bir "Table"a karşılık gelir. İyi bir `CREATE TABLE` senaryosunda bilmen gereken kritik bileşenler şunlardır:

#### 1. Temel Sözdizimi (Syntax) ve Örnek

Bir E-Ticaret sitesi için basit bir kullanıcı tablosu oluşturalım:

SQL

```
CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Username VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Age INT CHECK (Age >= 18),
    CreatedAt DATETIME DEFAULT GETDATE(),
    RoleId INT FOREIGN KEY REFERENCES Roles(Id)
);
```

#### 2. Constraints (Kısıtlamalar) - _Verinin Sigortası_

Tablo oluştururken koyduğun kurallara "Constraint" denir. Mülakatta "Veri bütünlüğünü (Data Integrity) nasıl sağlarsın?" sorusunun cevabı bunlardır:

- **PRIMARY KEY (PK):** Tablonun kimliğidir.
    
    - Her satırı benzersiz yapar.
        
    - Otomatik olarak o sütuna bir **Index** atar (Aramalar çok hızlanır).
        
    - Asla `NULL` olamaz.
        
- **IDENTITY(1,1) / AUTO INCREMENT:**
    
    - Sen veri eklerken ID vermezsin, SQL her yeni kayıtta sayıyı otomatik artırır (1, 2, 3...).
        
    - `(1,1)`: 1'den başla, 1'er 1'er artır demektir.
        
- **NOT NULL:**
    
    - "Bu alan boş geçilemez" kuralıdır. (Örn: Kullanıcı adı girmek zorunludur).
        
- **UNIQUE:**
    
    - Tekrara izin vermez. (Örn: Aynı e-posta ile ikinci kez kayıt olunamaz).
        
- **CHECK:**
    
    - Basit mantık kontrolleri yapar. (Örn: `Age >= 18` -> Yaşı 18'den küçük olanı kaydetme).
        
- **DEFAULT:**
    
    - Eğer veri gönderilmezse varsayılan bir değer atar. (Örn: Kayıt tarihi gönderilmezse, o anın tarihini (`GETDATE()`) bas).
        
- **FOREIGN KEY (FK):**
    
    - Başka bir tabloyla ilişki kurar. (Örn: `RoleId`, `Roles` tablosundaki bir ID'ye karşılık gelmelidir. Olmayan bir rolü kullanıcıya atayamazsın).
        

---

### Backend Developer İçin Neden Önemli?

1. **Code-First Yaklaşımı:** Sen .NET Core'da Entity Framework kullanırken C# sınıflarını yazarsın (`public class User { ... }`). EF Core, senin yerine arka planda bu `CREATE TABLE` kodunu üretir ve veritabanına gönderir. Ancak oluşan tablonun performanslı olması için C# tarafında doğru "Data Annotation"ları (`[Required]`, `[MaxLength(50)]`) kullanman gerekir. Bunların hepsi SQL'deki Constraint'lere dönüşür.
    
2. **Performansın Temeli:** Tabloyu oluştururken `PRIMARY KEY` belirlemek, verilerin disk üzerinde fiziksel olarak sıralanmasını sağlar (Clustered Index). Bunu yapmazsan veritabanı bir "yığın" (Heap) olur ve sorgular yavaşlar.
    
3. **İlişkisel Bütünlük:** `FOREIGN KEY` kullanmazsan, yanlışlıkla "Olmayan bir kategoride ürün" oluşturabilirsin. Bu da uygulamanın patlamasına sebep olur. SQL seviyesindeki koruma, kod seviyesindeki korumadan daha garantidir.