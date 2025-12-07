
### 1. Temel Soru: Neden Veritabanı? (File System vs. DBMS)

Bir yazılımda veriyi saklamak için neden basit bir `.txt` veya `.csv` dosyası kullanmıyoruz da devasa SQL sunucuları kuruyoruz? Bunu anlamadan veritabanının değerini anlayamayız.

Dosya sistemi (File System) ile veri tutmanın 4 ölümcül sorunu vardır:

1. **Redundancy (Veri Tekrarı):** Aynı öğrencinin adı 10 farklı dosyada geçer. Adı değişirse 10 dosyayı da güncellemen gerekir. Unutursan veri tutarsızlaşır (Inconsistency).
    
2. **Concurrency (Eşzamanlılık):** İki kişi aynı anda dosyayı açıp yazmaya çalışırsa ne olur? Dosya kilitlenir veya veriler birbirinin üzerine yazar (Race Condition).
    
3. **Security (Güvenlik):** Dosyaya erişimi olan herkes içini okuyabilir. "Sadece maaş sütununu gizle" diyemezsin.
    
4. **Data Integrity (Veri Bütünlüğü):** Yaş alanına "Yirmi Altı" yazılmasını engelleyemezsin. Veritabanı ise "Buraya sadece `INT` girebilir" diyerek kapıdaki güvenlik görevlisi olur.
    

İşte **DBMS (Database Management System)**, yani Veritabanı Yönetim Sistemi (SQL Server, PostgreSQL, Oracle vb.), bu sorunları çözen yazılım katmanıdır. Biz veriye doğrudan dokunmayız, DBMS'e emrederiz.

---

### 2. İlişkisel Veritabanları (RDBMS - Relational Database Management System)

1970'lerde E.F. Codd tarafından **Matematiksel Kümeler Teorisi (Set Theory)** üzerine kurulmuştur. Dünyadaki bankacılık, e-ticaret ve kurumsal sistemlerin %90'ı hala bunun üzerinde döner.

#### A. Yapı: Tablolar ve Şemalar (Schema)

RDBMS, veriyi **Tablolar (Tables)** içinde saklar.

- **Schema (Şema):** Veritabanının planıdır. Hangi tablolar var? Sütunların tipleri ne? (int, varchar, date). Bu yapı **katıdır**. Bir sütun eklemek maliyetlidir.
    
- **Row (Satır / Tuple):** Her bir veri kaydıdır.
    
- **Column (Sütun / Attribute):** Verinin özelliğidir.
    

#### B. İlişkiler ve Anahtarlar (The "Relational" Part)

Tablolar birbirine görünmez iplerle bağlıdır. Bu ipleri **Key (Anahtar)** sağlar.

1. **Primary Key (PK):** Bir satırın parmak izidir. Benzersiz (Unique) olmak zorundadır. Asla `NULL` olamaz. (Örn: `TCKimlikNo` veya `UserID`).
    
2. **Foreign Key (FK):** Başka bir tablodaki Primary Key'e işaret eden sütundur. İlişkiyi bu kurar.
    

**Örnek Senaryo:**

- `Users` tablosunda `ID: 1, Name: Ahmet` var.
    
- `Orders` tablosunda `OrderID: 101, UserID: 1` var.
    
- Buradaki `UserID: 1`, Users tablosuna giden bir **Foreign Key**'dir. Eğer Users tablosundan Ahmet'i silmeye çalışırsan, RDBMS seni durdurur: _"Hop! Bu adamın siparişleri var, silemezsin. Önce siparişleri sil."_ Buna **Referential Integrity** denir.
    

#### C. ACID Kuralları (Mülakatların Vazgeçilmezi)

İlişkisel veritabanlarının en büyük gücü **Transaction** (İşlem) yönetimidir. Bir işlemin güvenli olması için **ACID** standartlarına uyması gerekir:

1. **A - Atomicity (Ya Hep Ya Hiç):** Bir işlem 10 adımdan oluşuyorsa ve 9. adımda elektrik kesilirse, ilk 8 adım da geri alınır (Rollback). Yarım işlem olamaz. (Para benden çıktı ama sana gelmedi durumu olamaz).
    
2. **C - Consistency (Tutarlılık):** Veritabanı kurallarına (Constraints) aykırı veri girilemez.
    
3. **I - Isolation (İzolasyon):** Aynı anda 1000 kişi işlem yapsa bile, herkes sanki tek başına işlem yapıyormuş gibi hissetmelidir. Benim işlemim bitmeden sen benim sonucumu göremezsin.
    
4. **D - Durability (Dayanıklılık):** Sistem "Kaydedildi" dediyse, sunucu yansa bile o veri kaybolmaz (Diske yazma garantisi).
    

#### D. Ölçeklenme (Scaling)

RDBMS genelde **Dikey (Vertical) Scaling** kullanır. Yani performans yetmezse sunucunun RAM'ini, CPU'sunu artırırsın. Ama bunun bir donanımsal limiti vardır ve çok pahalıdır.

---

### 3. NoSQL Veritabanları ("Not Only SQL")

2000'lerin ortasında Google, Facebook, Amazon gibi devler ortaya çıkınca RDBMS yetmemeye başladı. Çünkü veri çok büyüktü (Big Data) ve çok hızlı değişiyordu.

NoSQL, katı kuralları (Schema) ve ACID özelliklerini biraz gevşeterek **Hız** ve **Esneklik** kazanır.

#### A. Türleri

Tek bir "NoSQL" yoktur, kullanım amacına göre 4 ana türü vardır:

1. Document Store (Örn: MongoDB):
    
    Veriyi JSON benzeri (BSON) formatta tutar. Tablo yoktur, "Collection" vardır. Satır yoktur, "Document" vardır.
    
    - _Avantajı:_ Şema yoktur. Bir dökümana "Yaş" eklerken diğerine eklemeyebilirsin. Yazılımcılar için çok rahattır.
        
    - _Kullanım:_ CMS, E-ticaret ürün katalogları.
        
    
    JSON
    
    ```
    // MongoDB Örneği
    {
      "id": 1,
      "ad": "Ahmet",
      "hobiler": ["Yüzme", "Kodlama"] // RDBMS'de bunu yapmak için ayrı tablo gerekirdi!
    }
    ```
    
2. Key-Value Store (Örn: Redis):
    
    En basit ve en hızlısıdır. Sadece bir anahtar ve bir değer vardır. Genelde veriyi RAM'de tutar (In-Memory).
    
    - _Kullanım:_ Caching (Önbellekleme), Oturum yönetimi.
        
3. Column-Family Store (Örn: Cassandra):
    
    Veriyi satır satır değil, sütun sütun saklar. Yazma hızı inanılmaz yüksektir.
    
    - _Kullanım:_ Facebook mesajları, IoT sensör verileri.
        
4. Graph Database (Örn: Neo4j):
    
    Veriyi "Düğümler" (Nodes) ve "İlişkiler" (Edges) olarak saklar.
    
    - _Kullanım:_ Sosyal ağlar (Arkadaşımın arkadaşını bul), Haritalar, Öneri motorları.
        

#### B. CAP Teoremi (Dağıtık Sistemlerin Anayasası)

Bir dağıtık sistemde (NoSQL genelde dağıtıktır), şu 3 özellikten **aynı anda sadece 2 tanesini** seçebilirsin:

1. **Consistency (Tutarlılık):** Herkes aynı anda aynı veriyi görür.
    
2. **Availability (Erişilebilirlik):** Sistem her zaman cevap verir (hata dönmez).
    
3. **Partition Tolerance (Bölünme Toleransı):** Ağ koparsa bile sistem çalışmaya devam eder.
    

- **RDBMS:** Genelde **CA** veya **CP**'dir. (Tutarlılık kutsaldır).
    
- **NoSQL:** Genelde **AP**'dir. (Tutarlılık sonradan gelebilir, yeter ki sistem hızlı cevap versin).
    

#### C. BASE Modeli (NoSQL'in Felsefesi)

ACID'in tam tersidir.

- **B - Basically Available:** Sistem her zaman cevap verir ama en güncel veriyi garanti etmez.
    
- **S - Soft State:** Verinin durumu zamanla değişebilir (replikasyon sürerken).
    
- **E - Eventual Consistency:** "Eninde sonunda" tüm sunucular aynı veriye sahip olacaktır. (Twitter'da bir tweet attığında arkadaşının bunu 5 saniye sonra görmesi sorun değildir. Bu Eventual Consistency'dir).
    

#### D. Ölçeklenme

NoSQL, **Yatay (Horizontal) Scaling** için tasarlanmıştır. Pahalı bir sunucu almak yerine, 10 tane ucuz sunucuyu yan yana bağlarsın (Sharding). Veriyi parçalayıp bu sunuculara dağıtır. Sınırsız büyüyebilir.

---

### Özet Karşılaştırma Tablosu

|**Özellik**|**RDBMS (SQL)**|**NoSQL**|
|---|---|---|
|**Veri Yapısı**|Tablolar, Satırlar, Sütunlar|Doküman, Key-Value, Graph|
|**Şema**|Katı (Schema-based)|Esnek (Schema-less)|
|**İlişkiler**|JOIN ile karmaşık ilişkiler|İlişkiler zayıftır, JOIN maliyetlidir|
|**Ölçeklenme**|Dikey (Vertical)|Yatay (Horizontal)|
|**Kural Seti**|ACID (Güvenlik Odaklı)|BASE (Hız/Erişim Odaklı)|
|**Örnekler**|PostgreSQL, SQL Server, MySQL|MongoDB, Redis, Cassandra|

---

