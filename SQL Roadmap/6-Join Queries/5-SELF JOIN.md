`SELF JOIN`, SQL dilinde özel bir komut (keyword) değildir; bir teknik adıdır. Bir tablonun, sanki iki farklı tabloymuş gibi **kendisiyle** birleştirilmesi işlemidir.

**Neden Kullanılır?** Genellikle **Hiyerarşik (Ağaç Yapısı)** verileri sorgulamak için kullanılır. En klasik örnek şirket hiyerarşisidir: Herkes "Personel" tablosundadır, ancak bazı personeller diğerlerinin yöneticisidir. "Ahmet'in yöneticisi kim?" sorusunun cevabı yine aynı tablodaki başka bir satırdadır. İşte bu ilişkiyi kurmak için tabloyu kendiyle joinleriz.

---

### Deep Dive: Aynadaki Yansıma ve Alias Zorunluluğu

Bir Junior Developer için kafa karıştırıcı olabilir: "Neden tabloyu kendisiyle çarpıştırıyoruz?" Mantığı şudur: SQL motoruna "Bu tabloyu bellekte kopyala, birine A de, diğerine B de ve bunları kıyasla" dersin.

#### 1. Alias (Takma İsim) Olmadan Asla!

Normal joinlerde tablo isimlerini kullanabilirsin ama Self Join'de **Alias kullanmak zorundasın**.

- **Hatalı:** `SELECT * FROM Employees JOIN Employees ON ...`
    
    - SQL hata verir: _"Hangi Employees tablosundan bahsediyorsun? İkisi de aynı isimde!"_
        
- **Doğru:** `FROM Employees E1 JOIN Employees E2`
    
    - Artık elimizde sanal olarak iki tablo var: `E1` (Çalışanlar) ve `E2` (Yöneticiler).
        

#### 2. Klasik Hiyerarşi Örneği

Bir `Employees` tablomuz olsun:

- **Id:** 1, **Name:** Ali, **ManagerId:** NULL (Patron)
    
- **Id:** 2, **Name:** Veli, **ManagerId:** 1 (Ali'ye bağlı)
    

Sorgu: "Çalışanların adını ve Yöneticilerinin adını yan yana getir."

SQL

```sql
SELECT 
    Calisan.Name AS Personel, 
    Yonetici.Name AS Amir
FROM Employees Calisan
LEFT JOIN Employees Yonetici ON Calisan.ManagerId = Yonetici.Id;
```

- Burada `LEFT JOIN` kullandık çünkü Patronun (Ali) yöneticisi yok (NULL). `INNER JOIN` yapsaydık patron listeden düşerdi (Self Join'de de `LEFT/INNER` ayrımı geçerlidir).
    

#### 3. Çift Kayıtları Bulma (Duplicate Finder) - _Mülakat Sorusu_

Mülakatta sorarlar: _"Tabloda aynı isme sahip ama farklı ID'si olan mükerrer kayıtları nasıl bulursun?"_ Cevap Self Join'dir.

SQL

```sql
SELECT A.Name 
FROM Users A
JOIN Users B ON A.Name = B.Name AND A.Id <> B.Id;
```

Mantık: İsimleri aynı olsun (`ON A.Name = B.Name`) AMA kendisi olmasın (`AND A.Id <> B.Id`).

---

### Backend Developer İçin Neden Önemli?

1. **Kategori Ağaçları (E-Ticaret):** "Elektronik -> Bilgisayar -> Laptop" yapısı genelde tek bir `Categories` tablosunda `ParentId` ile tutulur.
    
    - Backend tarafında (C#) "Laptop kategorisinin üst kategorisi (Parent) nedir?" diye sorduğunda, arka planda (veya EF Core ile) Self Join mantığı çalışır.
        
2. **Recursive (Özyinelemeli) İlişkiler - EF Core:** Entity Framework'te bir class kendi tipinde bir property barındırıyorsa bu Self Join'dir.
    
    C#
    
    ```csharp
    public class Employee {
        public int Id { get; set; }
        public string Name { get; set; }
    
        public int? ManagerId { get; set; }
        public Employee Manager { get; set; } // Kendine referans veriyor!
    }
    ```
    
    Bu yapıyı kurduğunda EF Core veritabanında `ManagerId` üzerinden Self Join ilişkisi kurar.
    
3. **Sosyal Ağlar:** Basit bir "Arkadaşlık" sistemi düşün. `Users` tablosu var. Arkadaşlıklar da yine bu tablodaki ID'ler arasında kurulur. "Arkadaşımın arkadaşlarını bul" sorgusu, zincirleme Self Join gerektirir.
    

Self Join, hiyerarşi ve ilişkisel veri modellemesinin (Recursive Relationship) temelidir. Bu konuyla birlikte **Join** başlığını tamamen bitirmiş olduk.