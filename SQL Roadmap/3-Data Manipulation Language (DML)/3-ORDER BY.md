`ORDER BY`, SQL sorgusunun sonucunda dönen listeyi belirli bir kurala göre (alfabetik, sayısal veya tarihsel) **sıralamak** için kullanılır.

**Neden Kullanılır?** Veritabanı tabloları, verileri fiziksel olarak karışık bir sırada (Heap) tutabilir. Eğer `ORDER BY` kullanmazsan, SQL Server sana verileri "kafasına göre" (genellikle o an diskten en hızlı okuduğu sırada) getirir. Bu sıra her sorguda değişebilir (Non-Deterministic). Kullanıcıya kararlı ve düzenli bir liste (örneğin "En yeni siparişler en üstte") göstermek için bu komut şarttır.

---

### Deep Dive: Sıralamanın Teknik Detayları ve Maliyeti

Bir Junior Developer olarak "Sırala gitsin" diyebilirsin ama büyük verilerde sıralama işlemi CPU ve RAM düşmanıdır. Mülakatta bu performans bilincini göstermelisin.

#### 1. Varsayılan ve Yönler

- **ASC (Ascending - Artan):** Varsayılandır. Yazmasan da çalışır.
    
    - Sayılar: 0 -> 9
        
    - Metin: A -> Z
        
    - Tarih: Eski -> Yeni
        
- **DESC (Descending - Azalan):** Tersten sıralar.
    
    - _Komut:_ `ORDER BY Price DESC` (En pahalı en üstte).
        

#### 2. Çoklu Sütun Sıralama (Hiyerarşi)

Sıralama tek bir sütunla sınırlı değildir.

- **Senaryo:** "Önce ülkeye göre sırala, aynı ülkedeki kullanıcıları da ismine göre sırala."
    
- **SQL:** `ORDER BY Country ASC, Name ASC`
    
- **Mantık:** SQL önce herkesi ülkeye göre dizer. Sadece "Turkey" olanları kendi içinde isme göre dizer. Öncelik soldan sağadır.
    

#### 3. NULL Değerlerin Konumu - _Mülakat Detayı_

Sıralama yaptığında `NULL` (boş) değerler nereye gider?

- SQL Server'da `NULL` değerler **en küçük** değer olarak kabul edilir.
    
- `ASC` sıralamada en başa gelirler.
    
- `DESC` sıralamada en sona giderler.
    

#### 4. Performans Maliyeti (Sorting Cost) - _Senior Dokunuşu_

Veritabanı için en pahalı işlemlerden biri sıralamadır.

- **Senaryo:** 1 milyon satırlık bir tabloda `ORDER BY CreatedDate` dedin.
    
- **Kötü Durum:** Eğer `CreatedDate` üzerinde bir **Index** yoksa; SQL tüm tabloyu belleğe (Memory/TempDB) yükler, orada sıralar ve sana verir. Bu işlem sunucuyu kasar.
    
- **İyi Durum:** Eğer `CreatedDate` üzerinde Index varsa; veriler zaten fiziksel veya mantıksal olarak sıralı tutulduğu için SQL ekstra işlem yapmaz, veriyi hemen verir.
    
- _Cevap:_ "Sık sıralama yapılan sütunlara Index atamalıyız."
    

#### 5. ORDER BY ve TOP/LIMIT İlişkisi

Sadece "En pahalı 5 ürünü getir" demek için `ORDER BY` zorunludur.

- `SELECT TOP 5 * FROM Products ORDER BY Price DESC`.
    
- Eğer `ORDER BY` yazmazsan, rastgele 5 ürün gelir; bu da iş mantığını bozar.
    

---

### Backend Developer İçin Neden Önemli?

1. **Pagination (Sayfalama):** E-ticaret sitelerinde "Sayfa 1, Sayfa 2" yapısı kurarken .NET'te `Skip(10).Take(10)` kullanırsın.
    
    - **Kural:** Bir veriyi sayfalamak için **mutlaka** sıralı olması gerekir.
        
    - Eğer `ORDER BY` kullanmadan sayfalama yaparsan, 1. sayfada gördüğün ürünü 2. sayfada tekrar görebilirsin. Veritabanı sırayı garanti etmez. Entity Framework bazı durumlarda `OrderBy` yoksa `Skip` kullanmana izin vermez ve hata fırlatır.
        
2. **LINQ Kullanımı:** C# tarafında:
    
    - `OrderBy(x => x.Name)` -> `ORDER BY Name ASC`
        
    - `OrderByDescending(x => x.Price)` -> `ORDER BY Price DESC`
        
    - `ThenBy(x => x.Age)` -> İkinci sütun sıralaması (SQL'deki virgül sonrası).
        
3. **Deterministik Sonuçlar:** Birim test (Unit Test) yazarken, testin her çalıştığında aynı sonucu vermesini beklersin. `ORDER BY` kullanmazsan testin bazen geçer bazen kalır (Flaky Test).