SQL veri tipleri, bir veritabanı tablosundaki sütunların hangi tür veriyi (sayı, metin, tarih, resim vb.) saklayabileceğini belirleyen kurallardır.

**Neden Kullanılır?** Her veriyi rastgele metin olarak saklayamayız. Veri tipleri; **Veri Bütünlüğünü** (Data Integrity) sağlar (örneğin "Yaş" sütununa "Ahmet" yazılmasını engeller), **Performansı** artırır (sayısal işlem yapmak metin işlemekten hızlıdır) ve **Depolama Alanını** optimize eder (gereksiz yere hafıza şişirilmez).

---

### Deep Dive: SQL Veri Tipleri ve .NET Karşılıkları

Sen bir .NET Developer adayı olduğun için, piyasada en çok karşılaşacağın **Microsoft SQL Server (T-SQL)** veri tiplerine odaklanacağız. Mülakatlarda en çok karıştırılan farkları (CHAR vs VARCHAR) bilmek zorundasın.

#### 1. Metin (String) Veri Tipleri - _En Çok Sorulan Kısım_

Burada 3 temel ayrım vardır: Uzunluk, Değişkenlik ve Unicode (Dil) desteği.

- **CHAR(n):** Sabit uzunlukludur.
    
    - _Örnek:_ `CHAR(10)` tanımladın ama içine "Ali" (3 harf) yazdın. Veritabanı bunu "Ali " (7 boşluk ekleyerek) saklar. 10 birim yer kaplar.
        
    - _Kullanım:_ TC Kimlik No, Telefon kodu gibi uzunluğu **kesinlikle** değişmeyen veriler için.
        
- **VARCHAR(n):** Değişken (Variable) uzunlukludur.
    
    - _Örnek:_ `VARCHAR(10)` tanımladın ve "Ali" yazdın. Sadece "Ali" kadar yer kaplar (+2 byte uzunluk bilgisi).
        
    - _Kullanım:_ İsim, Adres, E-posta.
        
- **NVARCHAR(n) - (National):** Unicode destekler.
    
    - _Farkı:_ `VARCHAR` sadece İngilizce karakterleri (ASCII) saklarken, `NVARCHAR` Türkçe (ş, ğ, ı), Çince, Arapça karakterleri saklayabilir. Karakter başına 2 kat yer kaplar ama global uygulamalar için zorunludur.
        
    - _NET Karşılığı:_ C# `string`.
        
- **TEXT / NTEXT:** _Eskidi (Deprecated)._ Artık kullanılmıyor, yerine `VARCHAR(MAX)` kullanılıyor.
    

#### 2. Sayısal (Numeric) Veri Tipleri

- **INT:** Tam sayılar. (–2 milyar ile +2 milyar arası).
    
    - _NET Karşılığı:_ `int` (Int32).
        
- **BIGINT:** Çok büyük tam sayılar.
    
    - _NET Karşılığı:_ `long` (Int64). YouTube izlenme sayıları veya Twitter ID'leri gibi devasa veriler için.
        
- **DECIMAL(p, s) / NUMERIC:** Virgüllü sayılar.
    
    - _Özelliği:_ Tam kesinlik (Precision) sağlar. Yuvarlama hatası yapmaz.
        
    - _Kullanım:_ **Para birimleri (Fiyat, Maaş)** için kesinlikle bu kullanılır. `FLOAT` kullanılmaz!
        
    - _NET Karşılığı:_ `decimal`.
        
- **FLOAT:** Bilimsel ve yaklaşık sayılar.
    
    - _Risk:_ Çok küçük kuruş farklarında hata yapabilir. Finansal veride kullanılmaz.
        

#### 3. Tarih ve Zaman (Date & Time)

- **DATETIME:** Tarih + Saat. (Örn: `2023-10-25 14:30:00`). Eski tiptir.
    
- **DATETIME2:** Daha hassas (milisaniye) ve daha az yer kaplayan modern versiyondur.
    
- **DATE:** Sadece tarih (Doğum günü gibi, saat önemsizse).
    
    - _NET Karşılığı:_ `DateTime`.
        

#### 4. Mantıksal ve Diğerleri

- **BIT:** SQL Server'da `BOOLEAN` (True/False) yoktur, yerine `BIT` vardır.
    
    - `0` = False, `1` = True.
        
    - _NET Karşılığı:_ `bool`.
        
- **GUID (UNIQUEIDENTIFIER):** Dünyada eşi benzeri olmayan rastgele uzun kodlar üretir (Örn: `6F9619FF-8B86-D011-B42D-00C04FC964FF`).
    
    - _NET Karşılığı:_ `Guid`.
        

---

### Backend Developer İçin Neden Önemli?

1. **C# ve SQL Uyumu (Mapping):** Sen C#'ta `decimal` olan bir "Ürün Fiyatı"nı veritabanında yanlışlıkla `INT` yaparsan, ürünün kuruşları (19.99 TL -> 19 TL) silinir. Veri kaybı yaşarsın. Entity Framework (ORM) kullanırken bu tiplerin birbirine doğru eşleşmesi hayati önem taşır.
    
2. **Performans:** Bir "Cinsiyet" alanı için `NVARCHAR(MAX)` kullanmak (ki sadece 'E' veya 'K' yazacaksın), veritabanını şişirir ve sorguları yavaşlatır. Doğru tip (`CHAR(1)` veya `BIT`), sorgunun milisaniyeler içinde bitmesini sağlar.
    
3. **Para Konusu:** Mülakatta "Parayı veritabanında hangi tipte tutarsın?" diye sorarlarsa, **"DECIMAL"** demezsen (Float dersen) elenirsin. Çünkü bankacılıkta kuruş hataları kabul edilemez.
    

Data Types konusu, veritabanı tasarımının alfabesidir.