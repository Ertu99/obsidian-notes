**QUERY OPTIMIZATION (Sorgu Optimizasyonu)**

Query Optimization, çalışan bir SQL sorgusunu **uçan** bir SQL sorgusuna dönüştürme sanatıdır. Bir Junior Developer "Çalışıyor mu? Çalışıyor" der ve bırakır. Senior Developer ise "Bu sorgu 1 milyon satırda sunucuyu patlatır mı?" diye sorar ve optimize eder.

**Neden Önemli?** 100 kayıtlı test veritabanında en kötü sorgu bile milisaniyede çalışır. Ancak gerçek hayatta (Production) milyonlarca veriyle çalışırken, optimize edilmemiş bir sorgu sitenin açılmamasına (Timeout) veya sunucunun kilitlenmesine sebep olur.

---

### Deep Dive: Performansın Altın Kuralları

Sorguları hızlandırmak için veritabanı motorunun (Engine) nasıl düşündüğünü anlaman gerekir. İşte mülakatlarda seni öne çıkaracak teknikler:

#### 1. `SELECT *` Günahı

Bunu asla yapma.

- **Hatalı:** `SELECT * FROM Users`
    
- **Doğru:** `SELECT Id, Name, Email FROM Users`
    
- **Neden?** İhtiyacın olmayan sütunları (örneğin kullanıcının Profil Resmi verisini - Base64) çekmek, hem Veritabanı CPU'sunu, hem Ağ trafiğini (Network I/O), hem de Backend uygulamasının RAM'ini gereksiz yere şişirir. "Sadece yiyeceğin kadarını al."
    

#### 2. SARGable (Search ARGument ABLE) Sorgular - _Pro İpucu_

Bu terim, bir sorgunun indeksi kullanıp kullanamayacağını belirtir. Eğer `WHERE` bloğunda sütun üzerinde fonksiyon çalıştırırsan, indeksi çöpe atarsın.

- **Kötü (Non-SARGable):** `WHERE YEAR(CreatedDate) = 2023`
    
    - SQL, her satır için tek tek `YEAR()` fonksiyonunu hesaplamak zorundadır. İndeksi kullanamaz (Full Table Scan).
        
- **İyi (SARGable):** `WHERE CreatedDate >= '2023-01-01' AND CreatedDate <= '2023-12-31'`
    
    - SQL, direkt tarih aralığına bakar ve indeksi kullanır.
        

#### 3. Wildcard (%) Kullanımı

Metin aramalarında `%` işaretinin yeri çok önemlidir.

- **Kötü:** `LIKE '%Ahmet%'` -> Başında ne olduğunu bilmediği için tüm tabloyu tarar. İndeks çalışmaz.
    
- **İyi:** `LIKE 'Ahmet%'` -> "A" harfiyle başladığını bildiği için indekste direkt A bölümüne gider. Çok hızlıdır.
    

#### 4. Execution Plan (İdam Planı Değil, Çalıştırma Planı)

SQL Management Studio'da veya diğer araçlarda sorguyu çalıştırmadan önce "Execution Plan" butonuna basarsın.

- Bu sana sorgunun röntgenini çeker.
    
- Eğer planda **"Table Scan"** veya **"Index Scan"** görüyorsan tehlike çanları çalıyor demektir. Hedefimiz **"Index Seek"** (Nokta atışı bulma) görmektir.
    

---

### Backend Developer İçin Neden Önemli?

1. **Timeout Hataları:** API'ye istek attığında dönmesi 30 saniyeyi geçiyorsa ve "Connection Timeout" alıyorsan, %99 ihtimalle arkada optimize edilmemiş, full table scan yapan bir sorgu vardır.
    
2. **EF Core `AsNoTracking()`:** Eğer veriyi sadece okuyup göstereceksen (üzerinde update yapmayacaksan), EF Core'da `.AsNoTracking()` metodunu kullanmalısın.
    
    - Bu, EF Core'un veriyi takip etme yükünü (Change Tracking overhead) kaldırır ve sorguyu ve RAM kullanımını ciddi oranda hafifletir.
        
3. **N+1 Problemi (Lazy Loading):** Backend tarafında döngü içinde veritabanına gitmek en büyük performans katilidir.
    
    - **Kötü:** Önce kullanıcıları çek, sonra `foreach` ile dönüp her birinin siparişlerini tek tek çek. (1000 kullanıcı = 1001 Sorgu).
        
    - **İyi (Eager Loading):** Kullanıcıları çekerken siparişlerini de `JOIN` (EF Core `.Include()`) ile tek seferde çek. (1 Sorgu).
        

Query Optimization, "Indexes" konusuyla kardeştir. Doğru indeks ve doğru yazılmış sorgu birleşince performans uçar.