`TRUNCATE TABLE`, bir veritabanı tablosundaki **tüm** kayıtları tek seferde silen, tabloyu ilk oluşturulduğu andaki tertemiz haline döndüren bir DDL (Data Definition Language) komutudur.

**Neden Kullanılır?** Milyonlarca satırlık bir "Log" tablosunu veya test verilerini temizlemek istediğinde `DELETE` komutu dakikalar, hatta saatler sürebilir ve veritabanını yorar. `TRUNCATE` ise bu işlemi saniyeler içinde yapar. Tabloyu silmez (DROP), sadece içini boşaltır.

---

### Deep Dive: TRUNCATE Nasıl Bu Kadar Hızlı Çalışır?

Önceki konuda `DELETE` ile farkına değinmiştik ama burada `TRUNCATE`'in arka plandaki "sihirli" çalışma mekanizmasına (User snippet'ında geçen "Extent Deallocation" olayına) odaklanacağız.

#### 1. Page ve Extent Deallocation (Sayfa İadesi)

Veritabanları verileri diskte "Page" (Sayfa) ve "Extent" (Sayfa grupları) denen bloklarda tutar.

- **DELETE komutu:** Satırları tek tek ziyaret eder, "Bu satırı siliyorum" diye işaretler ve bunu Transaction Log'a yazar. Milyon satır varsa milyon işlem yapar.
    
- **TRUNCATE komutu:** Satırlarla ilgilenmez. Veritabanı motoruna gider ve _"Bu tabloya ait olan tüm sayfaları (Pages) serbest bırak, artık boşlar"_ der.
    
    - **Analogy:** Bir kitaptaki yazıları silgiyle tek tek silmek (`DELETE`) ile o sayfaları koparıp atmak veya "Bu sayfalar artık boş sayılır" demek (`TRUNCATE`) arasındaki fark gibidir. Bu yüzden çok az Log üretir ve ışık hızındadır.
        

#### 2. Trigger'ları Tetiklemez (Bypassing Triggers)

Mülakatlarda sıkça sorulur: _"TRUNCATE yaparsam `ON DELETE` trigger'ı çalışır mı?"_

- **Cevap:** Hayır, çalışmaz.
    
- **Neden:** Triggerlar satır bazlı silme işlemlerinde tetiklenir. `TRUNCATE` satır silmediği (sayfa boşalttığı) için triggerlar devre dışı kalır. Eğer silinen veriyi başka bir yere yedekleyen bir triggerın varsa, `TRUNCATE` kullanmak veri kaybına yol açar.
    

#### 3. Foreign Key (Yabancı Anahtar) Kısıtlaması - _Kritik Engel_

Eğer senin `Users` tablon, başka bir tabloda (örneğin `Orders` tablosunda) Foreign Key ile referans gösteriliyorsa, SQL Server güvenlik gereği `Users` tablosunu **TRUNCATE yapmana izin vermez**. İçi boş olsa bile izin vermez.

- **Hata Mesajı:** _"Cannot truncate table because it is being referenced by a FOREIGN KEY constraint."_
    
- **Çözüm:** Önce `DELETE` kullanmalı ya da Foreign Key ilişkisini geçici olarak kaldırmalısın.
    

#### 4. Identity Reset (Kimlik Sıfırlama)

Tablondaki ID değeri en son 500.000'de kalmış olabilir.

- `TRUNCATE TABLE` çalıştırdığında, tablo sanki yeni yaratılmış gibi Identity değeri (Seed) başlangıç değerine (genelde 1'e) döner.
    

---

### Backend Developer İçin Neden Önemli?

1. **Test Otomasyonu:** .NET Core projelerinde entegrasyon testleri (Integration Tests) yazarken, her test senaryosundan önce veritabanının temiz olması gerekir. `DELETE` işlemi test süresini uzatır. `TRUNCATE` (veya `Respawn` gibi kütüphaneler) kullanarak test veritabanını milisaniyeler içinde sıfırlarsın.
    
2. **Log Temizliği:** Uygulamanın `ErrorLogs` tablosu zamanla GB'larca boyuta ulaşabilir. Bu tabloyu temizlemek için `DELETE` kullanırsan Transaction Log dosyan (LDF) şişer ve disk dolar. `TRUNCATE` bu iş için en güvenli ve performanslı yöntemdir.
    
3. **Performans Tuzağı:** Eğer bir tablonun triggerlarına güveniyorsan (örneğin "silinen kullanıcıyı `DeletedUsers` tablosuna at" gibi), `TRUNCATE` kullandığında bu kodların çalışmayacağını bilmek zorundasın. Yoksa veri tutarlılığını bozarsın.