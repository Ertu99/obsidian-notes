`LEFT JOIN` (veya `LEFT OUTER JOIN`), "Sol taraftaki tablonun **tamamını** al, sağ taraftaki tablodan ise sadece eşleşenleri getir" diyen birleştirme türüdür.

**Neden Kullanılır?** Bir okul müdürü olduğun düşün. "Tüm öğrencilerin listesini ve eğer varsa okula geldikleri servis plakasını getir" diyorsun.

- **INNER JOIN** yapsaydın: Servis kullanmayan (yürüyerek gelen) öğrencileri listeden silerdi.
    
- **LEFT JOIN** yaptığında: Tüm öğrencileri listeler. Servisi olmayanların servis plakası kısmına `NULL` (Boş) yazar. Veri kaybını önler.
    

---

### Deep Dive: "Sol" Taraf Neresi ve NULL Mantığı

Bir Junior Developer olarak `LEFT JOIN`'i anlamak kolaydır ama pratikte "Hangi tabloyu sola yazmalıyım?" karmaşası yaşanır.

#### 1. Sol ve Sağ Kavramı

SQL sorgusunda `LEFT JOIN` kelimesinden **önce** yazdığın tablo "SOL", **sonra** yazdığın tablo "SAĞ" tablodur.

- **Sorgu:** `FROM Users LEFT JOIN Orders`
    
- **Sol (Ana Tablo):** Users. (Buradaki herkes listelenecek).
    
- **Sağ (Yancı Tablo):** Orders. (Sadece varsa gelecek).
    

#### 2. NULL Üretimi (The Null Effect)

Eşleşme olmadığında ne olur?

- Ali'nin hiç siparişi yok. SQL, Ali'yi listeye ekler. Ancak `Orders` tablosundan gelmesi gereken `OrderDate`, `Amount` gibi sütunların içini neyle dolduracağını bilemez ve hepsine **NULL** basar.
    
- Yazılım tarafında (C#) bu veriyi karşılarken `NullReferenceException` yememek için o alanların "Nullable" (`int?`, `DateTime?`) olması gerekir.
    

#### 3. "Olmayanları Bulma" Tekniği (IS NULL) - _Mülakat Sorusu_

Mülakatta şunu sorarlar: _"Bana hiç sipariş vermemiş müşterileri getiren SQL sorgusunu yaz."_ Cevap `LEFT JOIN` ile verilir:

1. Kullanıcıları Siparişlerle LEFT JOIN yap. (Hepsini getir).
    
2. `WHERE SiparisId IS NULL` şartını koy.
    
3. Mantık: Eşleşenlerin sipariş ID'si doludur. Eşleşmeyenlerin (siparişi olmayanların) ID'si `LEFT JOIN` yüzünden `NULL` gelmiştir. Bunları filtreleyerek "Pasif Müşterileri" bulursun.
    

SQL

```sql
SELECT u.Name
FROM Users u
LEFT JOIN Orders o ON u.Id = o.UserId
WHERE o.Id IS NULL; -- İşte sihirli satır burası
```

---

### Backend Developer İçin Neden Önemli?

1. **LINQ `GroupJoin` ve `DefaultIfEmpty`:** C# LINQ ile `LEFT JOIN` yazmak biraz daha karışıktır.
    
    C#
    
    ```csharp
    var query = from u in context.Users
                join o in context.Orders on u.Id equals o.UserId into userOrders
                from subOrder in userOrders.DefaultIfEmpty() // Bu satır LEFT JOIN yapar
                select new { u.Name, OrderDate = subOrder != null ? subOrder.Date : (DateTime?)null };
    ```
    
    Buradaki `DefaultIfEmpty()`, "Eşleşme yoksa varsayılanı (NULL) getir" demektir.
    
2. **Raporlama Ekranları:** Bir admin panelinde kullanıcı listesi gösterirken, yanına "Son Sipariş Tarihi"ni yazdırmak istiyorsun. Siparişi olmayanı listeden atamazsın (INNER JOIN olmaz). Mecburen `LEFT JOIN` kullanıp, siparişi olmayanlara ekranda "-" veya "Sipariş Yok" yazdırmalısın.
    
3. **Master-Detail Yapılar:** Bir Blog sitesinde "Makaleler ve Yorumlar" ilişkisi. Henüz hiç yorum yapılmamış makaleyi de anasayfada göstermek zorundasın. Bu yüzden Makaleler (Sol) ile Yorumlar (Sağ) arasında `LEFT JOIN` kurulur.
    

`LEFT JOIN`, kapsayıcılık (Inclusivity) demektir. Ana tablodaki veriyi kaybetmek istemiyorsan adresin burasıdır.