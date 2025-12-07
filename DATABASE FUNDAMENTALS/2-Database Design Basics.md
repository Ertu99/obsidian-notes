
Bir backend developer için kod yazmaktan daha önemli bir şey varsa, o da veritabanını tasarlamaktır. Kötü bir C# kodunu 1 saatte refactor edip düzeltebilirsin. Ama kötü tasarlanmış, ilişkileri yanlış kurulmuş bir veritabanını düzeltmek aylar sürer, hatta bazen projeyi çöpe atıp baştan yazmayı gerektirir.

Bu konuyu, veritabanı mühendisliğinin kutsal kitabı sayılan **Normalizasyon (Normalization)** kuralları üzerinden, gerçek hayat senaryolarıyla inceleyeceğiz.

---

### 1. Neden Tasarım Yapıyoruz? (Anomaliler)

Rastgele tablo oluşturursak ne olur? Diyelim ki bir e-ticaret sitesi için tek bir "dev" tablo yaptık.

Hatalı Tablo (Siparisler):

| SiparisID | MusteriAdi | MusteriAdres | Urun | Fiyat | Kategori |

| :--- | :--- | :--- | :--- | :--- | :--- |

| 101 | Ahmet | İstanbul | Laptop | 20.000 | Elektronik |

| 102 | Ahmet | İstanbul | Mouse | 500 | Elektronik |

| 103 | Ayşe | Ankara | Klavye | 1.000 | Elektronik |

Bu tabloda 3 büyük **Anomali (Kusur)** vardır:

1. **Update Anomaly:** Ahmet İstanbul'dan İzmir'e taşındı. Sadece son siparişini güncellersen, eski siparişlerinde hala İstanbul yazar. Veri tutarsızlaşır (Inconsistency).
    
2. **Insert Anomaly:** Yeni bir ürün geldi (Monitör) ama henüz kimse satın almadı. Bu tabloya ekleyemezsin çünkü `SiparisID` ve `Musteri` bilgisi zorunlu. Ürünü sisteme sokamazsın.
    
3. **Delete Anomaly:** Ayşe tek siparişini iptal etti. O satırı silersen, "Klavye" ürününün fiyat bilgisini ve kategorisini de veritabanından tamamen silmiş olursun.
    

İşte **Normalizasyon**, bu sorunları çözmek için veriyi parçalara ayırma sanatıdır.

---

### 2. Normalizasyon Kuralları (Normal Forms)

Bir veritabanının sağlıklı olması için belirli seviyeleri (Forms) geçmesi gerekir. Genellikle **3. Normal Form (3NF)** endüstri standardıdır.

#### A. First Normal Form (1NF) - "Atomik Ol!"

Kural: Bir kutucuğa birden fazla veri tıkıştıramazsın. Her hücrede tek bir değer (Atomic Value) olmalı.

- **Hatalı:** `Hobiler: "Yüzme, Koşu, Sinema"` (Virgülle ayrılmış).
    
- **Doğru:** Her hobi için ayrı satır açmalısın veya ayrı tablo yapmalısın.
    
- **Neden?** "Yüzme sevenleri getir" diye sorgu atamazsın, performans ölür (`LIKE` kullanmak zorunda kalırsın).
    

#### B. Second Normal Form (2NF) - "Bütüne Sadık Ol!"

Kural: Tablodaki her sütun, tablonun **Primary Key'inin tamamına** bağlı olmalıdır. (Bu kural sadece Composite Key -Çoklu Anahtar- varsa geçerlidir).

- **Senaryo:** `SiparisDetay` tablosu (`SiparisID`, `UrunID`) ikilisini anahtar olarak kullanıyor.
    
- **Hatalı Sütun:** `UrunAdi`.
    
- **Analiz:** `UrunAdi`, `SiparisID`'ye bağlı değildir. Sadece `UrunID`'ye bağlıdır. Yani anahtarın sadece bir parçasına bağlıdır (Partial Dependency).
    
- **Çözüm:** `UrunAdi`nı buradan söküp `Urunler` tablosuna taşı.
    

#### C. Third Normal Form (3NF) - "Aracıları Kaldır!"

Kural: Bir sütun, anahtar olmayan başka bir sütuna bağlı olamaz (Transitive Dependency). "Arkadaşının arkadaşı benim arkadaşım değildir".

- **Senaryo:** `Siparisler` tablosu.
    
- **Sütunlar:** `SiparisID` (PK), `MusteriID`, `MusteriSehir`.
    
- **Hata:** `MusteriSehir`, `SiparisID`'ye mi bağlı? Hayır. Müşteriye bağlı. Müşteri değişirse şehir değişir.
    
- **Çözüm:** `MusteriSehir` verisini buradan sil, `Musteriler` tablosuna koy. Burada sadece `MusteriID` kalsın.
    

---

### 3. İlişki Türleri (Relationships)

Tabloları parçaladık (Normalize ettik). Şimdi bunları tekrar bağlamamız lazım (Foreign Key ile). 3 tür bağ vardır:

#### A. One-to-Many (Bire-Çok) - En Yaygını

Bir babanın birden çok çocuğu olabilir, ama çocuğun tek biyolojik babası vardır.

- **Örnek:** Bir `Müşteri` birden çok `Sipariş` verebilir.
    
- **Tasarım:** `Siparişler` tablosuna `MusteriID` (Foreign Key) eklenir.
    

#### B. Many-to-Many (Çoka-Çok) - "Junction Table" Zorunluluğu

Bir öğrenci birden çok ders alabilir. Bir dersi birden çok öğrenci alabilir.

- **Tasarım:** Bunu iki tabloyla çözemezsin. Araya üçüncü bir **Ara Tablo (Junction/Bridge Table)** gerekir.
    
    - `Ogrenciler` Tablosu
        
    - `Dersler` Tablosu
        
    - `OgrenciDersleri` Tablosu (`OgrenciID`, `DersID`) -> Burada kimin hangi dersi aldığı tutulur.
        

#### C. One-to-One (Bire-Bir) - Nadir

Bir vatandaşın sadece bir TC Kimlik Kartı olabilir.

- **Kullanım:** Genelde tablo çok şiştiğinde, az kullanılan verileri (örn: `Biyografi`, `CV`) ayırıp ana tabloyu hafifletmek için kullanılır.
    

---

### 4. Keys (Anahtarların Detayı)

- **Primary Key (PK):** Satırın kimliği. Eşsizdir. (Örn: `ID`).
    
- **Foreign Key (FK):** Başka tablonun kimliğine referans verir. İlişkiyi sağlar.
    
- **Composite Key:** Birden fazla sütunun birleşerek tek bir anahtar oluşturmasıdır.
    
    - _Örn:_ `OgrenciDersleri` tablosunda `OgrenciID` tek başına yetmez (aynı öğrenci başka ders alabilir), `DersID` tek başına yetmez. Ama `OgrenciID + DersID` ikilisi eşsizdir.
        

---

### 5. Denormalization (Kuralları Ne Zaman Yıkmalı?)

Mühendislik "Trade-off" (Ödünleşim) sanatıdır. Bazen performansı artırmak için 3NF kuralını bilerek bozarız. Buna **Denormalization** denir.

- **Senaryo:** E-ticaret sitesinde "Siparişlerim" sayfasını açıyorsun.
    
- **3NF Tasarımı:** Sistem gidip `Siparisler`, `SiparisDetay`, `Urunler`, `Kategoriler`, `Musteriler` tablolarının hepsini birleştirmek (JOIN) zorunda kalır. Milyonlarca kayıtta bu çok yavaştır.
    
- **Denormalize Çözüm:** `Siparisler` tablosuna `ToplamTutar` diye bir sütun ekleriz ve hesaplayıp oraya yazarız.
    
    - _Risk:_ Eğer ürün fiyatı değişirse burayı güncellemeyi unutmamalıyız (Data Consistency riski).
        
    - _Kazanç:_ Tek sorguda (JOIN yapmadan) sipariş listesi gelir. Okuma hızı uçar.
        

---

