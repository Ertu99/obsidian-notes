
# Konu 1: SQL Basics (SQL Temelleri ve Sorgu Mimarisi)

**SQL (Structured Query Language)**, İlişkisel Veritabanları (RDBMS) ile konuşmak için tasarlanmış standart dildir.

Bir C# geliştiricisi olarak bilmen gereken en önemli ayrım şudur:

- **C# (Imperative - Emir Kipi):** Bilgisayara işlemi **nasıl** yapacağını adım adım anlatırsın. (Döngü kur, if koy, listeye ekle...)
    
- **SQL (Declarative - Bildirimsel):** Bilgisayara **ne** istediğini söylersin, **nasıl** yapacağına karışmazsın.
    
    - _Sen:_ "Bana Ankara'da yaşayan müşterileri getir."
        
    - _SQL Engine:_ "Tamam, index taraması mı yapayım, tabloyu mu okuyayım, ben en hızlı yolunu bulurum."
        

### 1. SQL Dilinin Alt Kümeleri (Jargon Bilgisi)

SQL tek bir blok değildir. Mülakatlarda "DDL ile DML arasındaki fark nedir?" diye sorarlar.

1. **DQL (Data Query Language):** Veriyi çekmek için kullanılır.
    
    - Komut: `SELECT` (En çok bunu kullanacağız).
        
2. **DML (Data Manipulation Language):** Veriyi değiştirmek için kullanılır.
    
    - Komutlar: `INSERT`, `UPDATE`, `DELETE`.
        
3. **DDL (Data Definition Language):** Yapıyı (Tablo, Veritabanı) kurmak/değiştirmek için kullanılır.
    
    - Komutlar: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
        
4. **DCL (Data Control Language):** İzin ve yetki yönetimi.
    
    - Komutlar: `GRANT`, `REVOKE`.
        
5. **TCL (Transaction Control Language):** İşlem yönetimi (ACID).
    
    - Komutlar: `COMMIT`, `ROLLBACK`.
        

---

### 2. Logical Query Processing Order (Sorgu İşleme Sırası) - **Kritik Konu**

Çoğu yazılımcı SQL sorgusunu yazdığı sırayla çalıştığını sanır. **Bu yanlıştır.** SQL Motoru (Engine), sorguyu senin yazdığın sırayla okumaz.

**Senin Yazdığın Sıra (Lexical Order):**

SQL

```sql
1. SELECT (Ad, Soyad)
2. FROM (Personeller)
3. WHERE (Sehir = 'Istanbul')
4. ORDER BY (Ad)
```

**Motorun Çalıştırdığı Sıra (Logical Order):**

1. **FROM:** Önce hangi masaya (tabloya) gideceğini bulur. Veri kaynağını belirler.
    
2. **WHERE:** Tablodaki milyonlarca satırı filtreler. (Gereksiz veriyi belleğe almamak için en başta çalışır).
    
3. **GROUP BY:** Kalan veriyi gruplar.
    
4. **HAVING:** Gruplanmış veri üzerinde filtreleme yapar.
    
5. **SELECT:** Artık elinde kalan veriden hangi sütunları istediğini seçer.
    
6. **ORDER BY:** Sonuç setini sıralar.
    

Neden Önemli?

Şu hatayı yapmaman için:

SQL

```sql
SELECT (Maas * 1.18) AS BrutMaas 
FROM Personeller
WHERE BrutMaas > 20000 -- HATA!
```

**Bu kod çalışmaz.** Çünkü `WHERE` bloğu, `SELECT` bloğundan **önce** çalışır. Engine, `WHERE` satırına geldiğinde henüz `BrutMaas` diye bir şey hesaplanmamıştır (o daha sonraki SELECT aşamasında oluşacak).

---

### 3. Temel Operasyonlar (CRUD) ve Derin Detaylar

#### A. SELECT (Okuma)

Veri çekme komutudur.

- SELECT * (Yıldız) Tuzağı:
    
    Asla SELECT * FROM Users yazma.
    
    1. **Network Maliyeti:** İhtiyacın olmayan sütunları (örn: ProfileImage binary data) çekerek ağı tıkarsın.
        
    2. **Index Kullanımı:** Veritabanı, sadece istediğin sütunları getirmek için "Covering Index" kullanabilir. `*` dersen tabloyu komple okumak (Scan) zorunda kalır.
        
    
    - _Doğrusu:_ `SELECT Id, Name, Email FROM Users`
        
- **DISTINCT:** Tekrar eden satırları tekile indirir. (Pahalı bir işlemdir, gerekmedikçe kullanma).
    

#### B. INSERT (Ekleme)

Yeni kayıt atar.

SQL

```sql
INSERT INTO Users (Name, Email, Age) 
VALUES ('Ahmet', 'a@b.com', 26);
```

- **Identity/Auto Increment:** Genelde `ID` sütunu verilmez, veritabanı otomatik artırır.
    

#### C. UPDATE (Güncelleme) - Güvenlik Kilidi

Veriyi değiştirir.

SQL

```sql
UPDATE Users 
SET Age = 27 
WHERE Name = 'Ahmet';
```

- **Tehlike:** Eğer `WHERE` bloğunu unutursan, tablodaki **HERKESİN** yaşını 27 yaparsın. Geri dönüşü yoktur (Transaction açmadıysan).
    
- **Best Practice:** UPDATE yazarken önce `SELECT * FROM ... WHERE ...` yazıp doğru satırları seçtiğinden emin ol, sonra SELECT'i UPDATE'e çevir.
    

#### D. DELETE vs TRUNCATE (Silme)

İkisi de veri siler ama fark devasadır.

1. **DELETE:**
    
    - Satırları tek tek siler.
        
    - `WHERE` kullanabilirsin (`DELETE FROM Users WHERE Id=5`).
        
    - İşlem günlüğüne (Transaction Log) her satır için kayıt atar (Yavaştır).
        
    - Geri alınabilir (Rollback).
        
2. **TRUNCATE:**
    
    - Tabloyu sıfırlar (Fabrika ayarlarına döndürür).
        
    - `WHERE` **kullanamazsın**.
        
    - Satırları tek tek silmez, veri sayfalarını (Data Pages) topluca yok eder (Çok hızlıdır).
        
    - Otomatik artan ID sayacını (Identity) sıfırlar.
        

---

### 4. Null Handling (3 Değerli Mantık)

C#'ta `bool` ya `true` ya `false`tur. SQL'de ise 3 durum vardır:

1. True
    
2. False
    
3. **UNKNOWN (NULL)**
    

**NULL**, "Sıfır" veya "Boşluk" demek değildir. **"Bilinmiyor"** demektir.

- **Hata:** `WHERE Age = NULL`
    
    - Bilinmeyen bir şey, bilinmeyen bir şeye eşit olamaz. Sonuç `UNKNOWN` döner ve veri gelmez.
        
- **Doğru:** `WHERE Age IS NULL`
    

Bu yüzden matematiksel işlemlerde dikkatli olmalısın: `5 + NULL = NULL`. (Cebindeki parayı bilmiyorsam, üzerine 5 lira ekleyince sonucun ne olduğunu da bilemem).

---

### 5. Filtering ve Pattern Matching

Veriyi süzmek için `WHERE` içinde kullandığımız operatörler:

- `=`, `<>`, `>`, `<` (Standart karşılaştırma)
    
- `IN (1, 2, 3)`: Liste kontrolü. (OR, OR, OR yazmak yerine).
    
- `BETWEEN 10 AND 20`: Aralık kontrolü.
    
- `LIKE`: Metin arama.
    
    - `LIKE 'Ahmet%'`: Ahmet ile başlayanlar.
        
    - `LIKE '%Ahmet%'`: İçinde Ahmet geçenler. (Performans katilidir, Index kullanamaz).
        

---


SQL'de NULL bir değer değildir, "bilinmeyen" durumudur.

Veritabanı motoru o satıra geldiğinde şu mantıksal işlemi yapar:

1. **HR Satırı İçin:** `'HR' <> 'IT'` ? -> **TRUE**. (Bu satırı getir).
    
2. **IT Satırı İçin:** `'IT' <> 'IT'` ? -> **FALSE**. (Bu satırı getirme).
    
3. **NULL Satırı İçin:** `NULL <> 'IT'` ? -> **UNKNOWN**.
    

Kritik nokta şurası: `WHERE` bloğu sadece sonucu **TRUE** olanları kabul eder. **FALSE** veya **UNKNOWN** olanları içeri almaz.

Veritabanı der ki: _"Bu adamın departmanını bilmiyorum (NULL). Bilmediğim bir şeyin 'IT' olup olmadığını da bilemem. O yüzden riske girmem ve bu satırı getirmem."_

Doğrusu Nasıl Olmalıydı?

Eğer NULL olanları da istiyorsan, bunu açıkça belirtmelisin:

SQL

```sql
SELECT * FROM Personeller 
WHERE Departman <> 'IT' OR Departman IS NULL
```

