Foreign Key, iki tablo arasında **ilişki (Relationship)** kurmamızı sağlayan sütundur. Bir tablodaki bu anahtar, başka bir tablodaki **Primary Key**'i işaret eder.

**Neden Kullanılır?** Veritabanları "İlişkisel" (Relational) yapılardır. Bir E-ticaret sitesinde siparişler ve müşteriler ayrı tablolarda durur. Bir siparişin kime ait olduğunu bilmek için `Siparisler` tablosuna `MusteriId` adında bir sütun ekleriz. İşte bu sütun, bizi `Musteriler` tablosundaki gerçek kişiye götüren köprüdür.

---

### Deep Dive: İlişkilerin Kuralları (Referential Integrity)

Bir Junior Developer olarak Foreign Key'i sadece "tabloları bağlayan ip" olarak görebilirsin. Ancak veritabanı motoru için Foreign Key bir **güvenlik görevlisidir (Constraint)**.

#### 1. Referential Integrity (Referans Bütünlüğü)

Mülakatlarda bu terimi kullanmak çok şıktır. Şunu ifade eder:

- **Senaryo:** `Musteriler` tablosunda ID'si 1, 2, 3 olan kişiler var.
    
- **Kural:** Sen `Siparisler` tablosuna `MusteriId = 99` olan bir kayıt ekleyemezsin!
    
- **Neden?** Çünkü `Musteriler` tablosunda 99 numaralı kimse yok. Foreign Key, "Olmayan bir babaya çocuk eklenemez" kuralını zorla uygular. Bu sayede veritabanında **Öksüz Kayıtlar (Orphan Records)** oluşması engellenir.
    

#### 2. Cascade Kuralları (Zincirleme Reaksiyon) - _Mülakat Sorusu_

Bir Foreign Key tanımlarken, ana tablodaki (Parent) kayıt silindiğinde veya güncellendiğinde ne olacağını belirlersin.

- **NO ACTION / RESTRICT (Varsayılan):** Eğer Ahmet'in siparişi varsa, Ahmet'i sildirmez. Hata verir. En güvenlisidir.
    
- **ON DELETE CASCADE:** Ahmet silinirse, Ahmet'in tüm siparişlerini de otomatik siler. (Tehlikeli ama temizlik için kullanışlı).
    
- **ON DELETE SET NULL:** Ahmet silinirse, siparişleri silinmez ama `MusteriId` kısımları `NULL` (Boş) yapılır. "Sahibi bilinmeyen sipariş" olur.
    

#### 3. NULL Olabilir mi?

Evet, Foreign Key `NULL` olabilir.

- **Örnek:** `Personel` tablosunda `YoneticiId` alanı. En tepedeki patronun yöneticisi yoktur, o yüzden bu alan onda `NULL` olabilir. Bu, ilişkinin **zorunlu olmadığını** (Optional Relationship) gösterir.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core Navigation Properties:** C# tarafında kod yazarken:
    
    C#
    
    ```csharp
    public class Order {
        public int OrderId { get; set; }
        public int CustomerId { get; set; } // Foreign Key
        public Customer Customer { get; set; } // Navigation Property
    }
    ```
    
    EF Core, bu yapı sayesinde sen `order.Customer.Name` dediğinde arka planda Foreign Key üzerinden JOIN sorgusu atar ve müşterinin adını getirir. Foreign Key, nesne yönelimli programlamadaki (OOP) ilişkilerin veritabanı karşılığıdır.
    
2. **JOIN Performansı ve Indexleme - _Kritik İpucu_:**
    
    - **Primary Key** otomatik olarak Indexlenir (Hızlıdır).
        
    - **Foreign Key** otomatik olarak **INDEXLENMEZ!**
        
    - Eğer sürekli `JOIN ... ON s.MusteriId = m.Id` yapıyorsan ve `MusteriId` üzerinde Index yoksa, sorguların yavaşlar. Backend developer olarak FK olan sütunlara manuel olarak Index atmalısın.
        
3. **Veri Tutarlılığı:** Kod tarafında (C#) validation yazsan bile, veritabanı seviyesinde Foreign Key kullanmazsan, yanlışlıkla veritabanına manuel müdahale edildiğinde sistem patlar. FK, son savunma hattıdır.