DDL (Veri Tanımlama Dili), SQL'in veritabanındaki verilerle değil, o verilerin tutulacağı **yapılarla (Structure/Schema)** ilgilenen alt kümesidir.

**Neden Kullanılır?** Bir eve eşya (veri) yerleştirmeden önce evin duvarlarını örmeli, odalarını belirlemelisin. DDL, bu inşaat işini yapar. Tablo oluşturmak, bir tabloya yeni sütun eklemek, veritabanını tamamen silmek veya bir index tanımlamak için kullanılır. Özetle; veritabanının iskeletini oluşturur ve yönetir.

---

### Deep Dive: Temel DDL Komutları ve Kritik Farklar

DDL komutları veritabanı yapısını **kalıcı** olarak değiştirir. Çoğu veritabanında DDL komutları çalıştırıldığı anda "Auto-Commit" olur, yani geri alınması (Rollback) zordur veya imkansızdır.

#### 1. CREATE (Oluşturma)

Sıfırdan bir veritabanı nesnesi (Tablo, View, Index, Stored Procedure) oluşturur.

- **Örnek:** `CREATE TABLE Users (Id INT, Name VARCHAR(50))`
    
- _Mülakat İpucu:_ Sadece tablo değil, **Index** oluşturmak da (performans için) bir DDL işlemidir.
    

#### 2. ALTER (Değiştirme)

Var olan bir yapıyı bozmadan üzerinde tadilat yapar.

- **Kullanım:** Tabloya yeni bir kolon eklemek (`ADD Column`), kolonun tipini değiştirmek (`ALTER COLUMN`) veya bir kısıtlamayı (Constraint) kaldırmak için kullanılır.
    
- _Risk:_ Canlı sistemde (Production) içi dolu bir tabloya `ALTER` çekmek, tabloyu kilitleyebilir (Lock) veya veri tipini değiştirirken veri kaybına (Data Truncation) yol açabilir.
    

#### 3. DROP (Silme/Yok Etme)

Bir nesneyi veritabanından tamamen, kökünden siler.

- **Örnek:** `DROP TABLE Users`.
    
- **Sonuç:** Tablo da gider, içindeki veriler de gider, tabloya bağlı indexler de gider. Geri dönüşü yoktur.
    

#### 4. TRUNCATE (Sıfırlama/Boşaltma) - _Mülakatın En Büyük Tuzağı_

Tablonun yapısını (duvarlarını) korur ama içindeki tüm verileri (eşyaları) tek seferde siler.

- **Örnek:** `TRUNCATE TABLE Users`.
    

---

### 🔥 Mülakat Sorusu: TRUNCATE vs DELETE Farkı Nedir?

Junior developer mülakatlarının vazgeçilmez sorusudur. İkisi de veriyi siler ama çalışma mantıkları tamamen farklıdır.

1. **Tür Farkı:** `TRUNCATE` bir **DDL** komutudur (Yapıyı sıfırlar). `DELETE` bir **DML** (Data Manipulation Language) komutudur (Veriyi manipüle eder).
    
2. **Hız:**
    
    - `DELETE`, satırları teker teker siler ve her silme işlemini Log dosyasına yazar. Çok yavaştır.
        
    - `TRUNCATE`, tablonun veri sayfalarını (Data Pages) toptan serbest bırakır. Çok az Log tutar. Çok **hızlıdır**.
        
3. **Identity (ID) Sıfırlama:**
    
    - `DELETE` ile tüm kayıtları silsen bile, sayaç (Identity) kaldığı yerden devam eder (Örn: En son ID 100 ise, yeni kayıt 101 olur).
        
    - `TRUNCATE` sayacı **sıfırlar**. Yeni kayıt ID 1'den başlar.
        
4. **WHERE Koşulu:**
    
    - `DELETE` komutunda `WHERE ID = 5` diyebilirsin.
        
    - `TRUNCATE` komutunda koşul veremezsin. Ya hepsi silinir ya hiçbiri.
        

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core Migrations:** Sen .NET Core'da "Code First" çalışırken `Add-Migration` ve `Update-Database` komutlarını kullandığında, EF Core arka planda senin C# sınıflarını (Class) SQL **DDL** komutlarına (`CREATE TABLE`, `ALTER TABLE`) çevirir ve veritabanına uygular. Migration dosyalarını okuduğunda bu komutları göreceksin.
    
2. **Test Ortamları:** Yazdığın birim testlerde (Unit Tests) veya entegrasyon testlerinde, her testten önce veritabanını temizlemek istersin. Burada `DELETE` kullanmak testi yavaşlatır, `TRUNCATE` kullanmak testi uçurur.
    
3. **Deployment (Canlıya Çıkış):** Uygulamanın yeni versiyonunu çıkarken veritabanı şemasını değiştirmen gerekebilir. Hangi DDL komutunun tabloyu kilitleyip (Lock) siteyi erişilmez hale getireceğini bilmelisin. (Örneğin `ALTER TABLE` büyük tablolarda risklidir).