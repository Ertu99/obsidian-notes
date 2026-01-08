**Primary Key (Birincil Anahtar)**

Primary Key, bir veritabanı tablosundaki her bir satırı **benzersiz (unique)** kılan ve kimliğini belirleyen sütundur. Türkiye Cumhuriyeti vatandaşları için "TC Kimlik No" neyse, tablodaki veriler için Primary Key odur.

**Neden Kullanılır?** Veritabanında "Ahmet Yılmaz" isminde binlerce kişi olabilir. Eğer Ahmet'in bilgilerini güncellemek istersen, hangi Ahmet olduğunu bilgisayara nasıl anlatacaksın? İşte bu karışıklığı önlemek için her kayıt benzersiz bir kimliğe (Genellikle `ID` sütunu) sahip olmalıdır.

---

### Deep Dive: Sadece Bir ID Değil, Performansın Kalbidir

Bir Junior Developer, Primary Key'i sadece "Benzersiz numara" olarak bilir. Ama mülakatta seni öne geçirecek olan detay, onun **fiziksel depolama** üzerindeki etkisidir.

#### 1. Clustered Index (Kümelenmiş İndeks) Etkisi - _Mülakatın Zirvesi_

SQL Server'da bir tabloya Primary Key atadığın anda, arka planda otomatik olarak **Clustered Index** oluşturulur.

- **Ne demek?** Veritabanı, verileri diske yazarken Primary Key sırasına göre **fiziksel olarak sıralar**.
    
- **Analogy:** Bir telefon rehberi düşün. İsimler A'dan Z'ye sıralıdır. Aradığın kişiyi bulmak saniyeler sürer. Eğer Primary Key (Clustered Index) olmasaydı, veriler diskte rastgele bir yığın (Heap) halinde dururdu; "Ahmet"i bulmak için tüm rehberi tek tek gezmek gerekirdi.
    
- **Sonuç:** Primary Key, sorguların ışık hızında çalışmasını sağlar.
    

#### 2. Kurallar Üçlüsü

Primary Key olabilmek için bir sütunun şu 3 şartı sağlaması gerekir:

1. **Unique (Benzersiz):** Aynı ID'den iki tane olamaz. (Örn: İki tane 5 numaralı kayıt olamaz).
    
2. **Not Null:** Boş bırakılamaz. Kimliksiz kayıt olamaz.
    
3. **Single:** Bir tabloda sadece **1 tane** Primary Key tanımı olabilir. (Ancak bu anahtar birden fazla sütunun birleşiminden oluşabilir -> Composite Key).
    

#### 3. Surrogate Key vs. Natural Key

Mülakatta sorarlar: _"TC Kimlik No mu ID olsun, yoksa otomatik artan sayı mı?"_

- **Natural Key (Doğal Anahtar):** Verinin içinde zaten var olan benzersiz değer (Örn: TC No, E-posta, Barkod).
    
    - _Sorun:_ E-posta değişebilir. TC No değişmez ama veritabanı performansında metin (string) indexlemek yavaştır.
        
- **Surrogate Key (Yapay Anahtar):** Veriyle ilgisi olmayan, sistemin ürettiği anlamsız sayılardır (1, 2, 3...).
    
    - _Tercih Edilen:_ Backend dünyasında standart budur. Genellikle `INT IDENTITY(1,1)` veya `GUID` kullanılır. İş süreçleri değişse bile ID değişmez.
        

#### 4. Composite Primary Key (Bileşik Anahtar)

Bazen tek bir sütun benzersizliği sağlamaya yetmez.

- **Örnek:** Bir öğrenci (OgrenciId) bir dersten (DersId) sadece bir kez not alabilir.
    
- Burada `OgrenciId` tek başına PK olamaz (Öğrenci başka ders de alabilir). `DersId` de olamaz.
    
- **Çözüm:** `PK (OgrenciId + DersId)`. İkisinin kombinasyonu benzersizdir.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core (Convention):** .NET'te `User` adında bir class oluşturduğunda, içine `Id` veya `UserId` adında bir property koyarsan, EF Core bunu otomatik olarak Primary Key algılar ve veritabanını ona göre kurar.
    
    C#
    
    ```csharp
    public class User {
        public int Id { get; set; } // Otomatik PK olur
        public string Name { get; set; }
    }
    ```
    
2. **İlişkilerin Çapası (Anchor):** Bir sonraki konu olan **Foreign Key**'in çalışabilmesi için, bağlandığı tabloda mutlaka bir Primary Key olmak zorundadır. PK olmadan, tablolar arasında ilişki (Join) kuramazsın. İlişkisel veritabanı (RDBMS) mantığı çöker.
    
3. **GUID vs INT Tartışması:** .NET projelerinde bazen `INT` yerine `Guid` (Global Unique Identifier) kullanılır.
    
    - `INT`: Okuması kolay, az yer kaplar (4 byte), sıralıdır.
        
    - `GUID`: Tahmin edilemez (Güvenli), birleştirilmesi kolay, çok yer kaplar (16 byte), düzensizdir (Index parçalanması yapar).
        
    - _Mülakat Cevabı:_ "Dağıtık sistemlerde (Microservices) çakışma olmaması için GUID, monolitik ve performans odaklı sistemlerde INT tercih ederim."