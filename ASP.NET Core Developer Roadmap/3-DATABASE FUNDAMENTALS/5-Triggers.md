
Trigger'lar, veritabanının "gizli ajanlarıdır" veya savaş filmlerindeki "mayızlar" gibidir. Sen fark etmeden, belirli bir olay gerçekleştiğinde (örneğin biri bir tabloya veri girdiğinde) otomatik olarak patlar (çalışır).

Mühendislik açısından Trigger'lar hem **muazzam bir güç** hem de **büyük bir risk** kaynağıdır. Çoğu kıdemli yazılımcı "Zorunda kalmadıkça Trigger kullanma" der. Nedenini ve nasıl çalıştığını derinlemesine inceleyelim.

---

### 1. Trigger Nedir ve Nasıl Çalışır? (Mekanik)

Metinde dendiği gibi, Trigger'lar "özel bir Stored Procedure türüdür". Ama farkı şudur: Stored Procedure'ü sen çağırırsın (`EXEC sp_Islem`), Trigger ise kendi kendine çalışır.

Trigger'lar, onları tetikleyen işlemle (örneğin `INSERT`) **aynı Transaction (İşlem)** içinde çalışır.

- **Kritik Bilgi:** Eğer Trigger hata verirse, onu tetikleyen `INSERT` işlemi de iptal olur (Rollback). Yani Trigger, işlemin ayrılmaz bir parçası haline gelir.
    

---

### 2. Sihirli Tablolar: `inserted` ve `deleted` (Magic Tables)

Bir Backend Developer olarak bilmen gereken en önemli teknik detay budur. Trigger çalıştığı o milisaniyelik anda, RAM'de (hafızada) geçici olarak iki sanal tablo oluşur:

1. **`inserted` Tablosu:**
    
    - `INSERT` işleminde: Eklenen yeni verinin kopyası buradadır.
        
    - `UPDATE` işleminde: Satırın **yeni (güncellenmiş)** hali buradadır.
        
2. **`deleted` Tablosu:**
    
    - `DELETE` işleminde: Silinen verinin kopyası buradadır.
        
    - `UPDATE` işleminde: Satırın **eski (değişmeden önceki)** hali buradadır.
        

**Mühendislik Notu:** `UPDATE` aslında veritabanı için "Eski kaydı sil (`deleted`), yeni kaydı ekle (`inserted`)" işlemidir. Bu yüzden Update trigger'larında her iki tabloyu da kullanırız.

---

### 3. Kullanım Alanları (Ne Zaman Kullanmalı?)

Trigger'ları iş mantığı (Business Logic) için kullanmak modern dünyada pek sevilmez. Ancak şu senaryolar için rakipsizdir:

#### A. Audit Logging (Denetim Kaydı) - **En Yaygın Kullanım**

"Maaş tablosunu kim, ne zaman, hangi değerden hangi değere değiştirdi?" sorusunun cevabını C# tarafında tutmak zordur (biri DB'ye doğrudan girip değiştirirse yakalayamazsın).

Trigger ile bunu veritabanı seviyesinde yakalarsın.

SQL

```sql
CREATE TRIGGER trg_MaasTakip
ON Personeller
AFTER UPDATE
AS
BEGIN
    -- Sadece Maaş sütunu değiştiyse çalış
    IF UPDATE(Maas)
    BEGIN
        INSERT INTO MaasLoglari (PersonelID, EskiMaas, YeniMaas, Degistiren, Tarih)
        SELECT 
            i.ID, 
            d.Maas,  -- deleted tablosundan ESKİ maaşı al
            i.Maas,  -- inserted tablosundan YENİ maaşı al
            SYSTEM_USER,
            GETDATE()
        FROM inserted i
        JOIN deleted d ON i.ID = d.ID;
    END
END
```

#### B. Complex Constraints (Karmaşık Kısıtlamalar)

`CHECK` constraint basit kurallar içindir (`Maas > 0`). Ama "Bir personel aynı anda 3 projeden fazlasında çalışamaz ve eğer 3. proje 'Gizli' ise ek onay gerekir" gibi karmaşık, başka tabloları sorgulayan kuralları Trigger ile yazarsın.

---

### 4. Trigger Türleri (AFTER vs INSTEAD OF)

1. AFTER Triggers (Varsayılan):
    
    İşlem (Insert/Update) biter, kayıt tabloya yazılır, SONRA trigger çalışır.
    
    - _Kullanım:_ Loglama, mail atma, başka tabloyu güncelleme.
        
2. INSTEAD OF Triggers (Yerine):
    
    İşlem (Insert/Update) engellenir, YERİNE trigger çalışır. Kayıt tabloya yazılmaz.
    
    - _Kullanım:_ Genellikle **View**'lar üzerinde kullanılır. Normalde View'lara insert yapılamaz ama INSTEAD OF trigger ile gelen veriyi alıp arka plandaki gerçek tablolara dağıtabilirsin.
        

![SQL trigger execution flow diagram resmi](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcS4jGQ3bFw7EeAgNo4bI2Fy53hE4GTdgOS-IXhLR-CXg2_rJ6YSuIzaIdnZuObNok9fnOFzlYC9wiY5JMH0h04U6YfJHHiaMhAaYT0lPV1Loaog1Ng)

Shutterstock

---

### 5. Karanlık Taraf: Neden Trigger Sevilmez?

Bir yazılım mimarı olarak Trigger kullanmadan önce iki kere düşünmelisin.

1. Görünmezlik (Invisibility):
    
    Yeni işe başlayan bir yazılımcı C# koduna bakar: UPDATE Users SET Status = 'Active'. Kodu çalıştırır. Arka planda Trigger devreye girer, mail atar, log yazar, SMS servisine gider... Yazılımcının bunlardan haberi yoktur. Debug etmesi (hata ayıklaması) kabustur.
    
2. Performans (Silent Killer):
    
    Basit bir INSERT işlemi, arkada çalışan ağır bir Trigger yüzünden 5 saniye sürebilir. Sorunu bulmak zordur.
    
3. Recursive Triggers (Özyineleme):
    
    Tablo A'daki Trigger, Tablo B'yi günceller. Tablo B'deki Trigger dönüp Tablo A'yı günceller. Sonsuz döngü! (SQL Server bunu 32 adımda keser ama yine de tehlikelidir).
    

---

