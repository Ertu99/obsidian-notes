Index, veritabanı performansının **sihirli değneğidir**. Bir tablodaki verilere erişimi inanılmaz derecede hızlandıran, veritabanı motoruna "Aradığın veri tam olarak şu rafta" diye yol gösteren yardımcı yapılardır.

**Neden Kullanılır?** 500 sayfalık bir ders kitabında "Polimorfizm" konusunu aradığını düşün.

- **İndeks Yoksa:** 1. sayfadan başlar, tek tek tüm sayfaları çevirerek kelimeyi ararsın (Full Table Scan). Çok yavaştır.
    
- **İndeks Varsa:** Kitabın arkasındaki "İndeks" bölümünü açarsın, "P" harfine bakarsın, sayfa numarasını bulur ve direkt o sayfayı açarsın (Index Seek). Saniyeler sürer.
    

---

### Deep Dive: Hızın Bedeli (Trade-off)

Bir Junior Developer, "Madem bu kadar hızlı, neden her sütuna indeks koymuyoruz?" diye sorabilir. Mülakatlarda seni tam olarak buradan vururlar.

#### 1. Okuma Hızı vs. Yazma Hızı

- **SELECT (Okuma):** İndeksler okumayı uçurur.
    
- **INSERT/UPDATE/DELETE (Yazma):** İndeksler yazmayı **yavaşlatır**.
    
- **Neden?** Çünkü sen tabloya yeni bir kayıt eklediğinde, veritabanı sadece tabloya yazmakla kalmaz; gider o tabloya bağlı olan indeksleri de (kitabın arkasındaki listeyi de) günceller, yeniden sıralar.
    
- **Kural:** Çok okunan (Read-Heavy) tablolara indeks koyulur, çok yazılan (Write-Heavy - örn: Log tabloları) tablolarda indeks minimumda tutulur.
    

#### 2. Full Table Scan (Kâbus)

İndeks olmayan bir sütunda (örn: `TelefonNo`) arama yaparsan (`WHERE TelefonNo = '555...'`), SQL motoru tablodaki 1 milyon satırı tek tek gezmek zorundadır. Sunucu CPU'su %100 olur, fanlar dönmeye başlar. İndeks bu taramayı engeller.

#### 3. Seçicilik (Selectivity)

Her sütuna indeks konmaz.

- **Kötü Aday:** `Cinsiyet` (Sadece Kadın/Erkek var). İndeks koysan da tablonun yarısını getireceği için veritabanı indeksi kullanmaz, yine tarama yapar.
    
- **İyi Aday:** `TCKimlikNo`, `Email`, `SiparisNo`. Neredeyse her satırda farklı değerler vardır (Yüksek Seçicilik).
    

---

### Backend Developer İçin Neden Önemli?

1. **API Performansı:** Backend tarafında yazdığın bir API 5 saniyede cevap veriyorsa, %90 ihtimalle sorguladığın sütunda (WHERE şartındaki alanda) indeks yoktur. Bir `CREATE INDEX` komutuyla süreyi 5 saniyeden 50 milisaniyeye düşürebilirsin. Kodda hiçbir değişiklik yapmadan!
    
2. **Entity Framework Core:** Veritabanı tasarımını kodla yapıyorsan (Code-First), indeksleri şöyle tanımlarsın:
    
    C#
    
    ```csharp
    modelBuilder.Entity<User>()
        .HasIndex(u => u.Email); // Email üzerinde arama hızlı olsun
    ```
    
3. **Deadlock (Kilitlenme) Riski:** Gereksiz yere çok fazla indeks koyarsan, güncelleme işlemleri uzayacağı için tablolar daha uzun süre kilitli kalır ve sistemde "Deadlock" hataları artar.
    

İndekslerin genel mantığı, "Hız vs Kayıt Maliyeti" dengesi üzerine kuruludur.