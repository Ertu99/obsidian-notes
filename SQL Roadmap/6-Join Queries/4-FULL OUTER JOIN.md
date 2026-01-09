`FULL OUTER JOIN` (kısaca `FULL JOIN`), SQL birleştirmelerinin "Hepsini Getir" komutudur. Hem `LEFT JOIN`'in hem de `RIGHT JOIN`'in yaptığı işi aynı anda yapar. İki tablodaki **tüm** kayıtları getirir; eşleşenleri yan yana yazar, eşleşmeyenlerin ise karşısına `NULL` basar.

**Neden Kullanılır?** İki farklı listeyi karşılaştırıp "Kimler ortak, kimler sadece solda var, kimler sadece sağda var?" sorusunun cevabını tek seferde almak için kullanılır. Genellikle veri tutarsızlıklarını bulmak veya iki sistemi senkronize etmek (Reconciliation) için hayati önem taşır.

---

### Deep Dive: Kaosun İçindeki Düzen

Bir Junior Developer olarak `FULL JOIN`'i kod içinde çok sık kullanmazsın (genelde raporlama ve bakım işlerinde kullanılır). Ancak mülakatta "İki tablo arasındaki farkları nasıl bulursun?" sorusunun cevabı budur.

#### 1. Çalışma Mantığı

İki kümenin birleşimi (Union) gibidir.

- **Senaryo:** `MuhasebeKayitlari` ve `BankaHareketleri` tablolarını karşılaştırıyorsun.
    
- **Durum 1 (Eşleşen):** Hem muhasebede hem bankada varsa yan yana gelir. (Her şey yolunda).
    
- **Durum 2 (Sadece Sol):** Muhasebede var ama Bankada yoksa? Banka sütunları `NULL` gelir. (Belki ödeme bankadan düşmedi?).
    
- **Durum 3 (Sadece Sağ):** Bankada var ama Muhasebede yoksa? Muhasebe sütunları `NULL` gelir. (Belki muhasebeci kaydetmeyi unuttu?).
    

#### 2. MySQL İstisnası - _Mülakat Detayı_

Her veritabanı motoru `FULL JOIN`'i desteklemez.

- **SQL Server, PostgreSQL, Oracle:** Destekler.
    
- **MySQL:** Desteklemez! MySQL mülakatında "FULL JOIN yap" derlerse; _"MySQL'de doğrudan FULL JOIN yoktur. Bunu LEFT JOIN ve RIGHT JOIN yapıp sonuçları UNION (Birleştirme) ile bağlayarak simüle ederim"_ demelisin.
    

#### 3. WHERE ile Filtreleme (Exclusive Full Join)

Eğer sadece "Birbirinden farklı olanları" (Ortak olmayanları) görmek istiyorsan:

SQL

```sql
SELECT * FROM TableA A
FULL OUTER JOIN TableB B ON A.Id = B.Id
WHERE A.Id IS NULL OR B.Id IS NULL;
```

Bu sorgu, her iki tarafın "Farklarını" döker.

---

### Backend Developer İçin Neden Önemli?

1. **LINQ Desteği Yoktur:** .NET ve C# dünyasında `FullOuterJoin` diye hazır bir LINQ metodu **yoktur**.
    
    - Bunu kodda yapmak istersen performansı çok zorlarsın (iki listeyi belleğe çekip kıyaslamak gerekir). Bu yüzden bu tür işleri C# tarafında değil, Stored Procedure yazarak SQL tarafında yapmak en doğrusudur.
        
2. **Maliyet (Performans):** `FULL JOIN`, veritabanı için pahalı bir işlemdir. İki tabloyu da baştan sona okur, eşleştirir, boşlukları doldurur. Canlı ve yoğun trafikli bir sayfada (örneğin Anasayfa akışı) `FULL JOIN` kullanmak sunucuyu yorabilir. Genellikle gece çalışan "Mutabakat Job"larında kullanılır.
    
3. **Veri Göçü (Data Migration):** Eski veritabanından yeni veritabanına veri taşırken, "Hangi veriler aktarılamadı?" kontrolü için `FULL JOIN` hayat kurtarır.