Correlated Subquery, SQL dünyasının **"Foreach Döngüsü"**dür. Normal bir subquery tek başına çalışıp bir sonuç üretirken, correlated subquery **dışarıdaki sorgunun her bir satırı için** tekrar tekrar çalışır.

**Neden Kullanılır?** Genellikle "Bir veriyi, ait olduğu grubun geneliyle kıyaslamak" için kullanılır.

- _Örnek:_ "Her ürünü getir ama sadece fiyatı **kendi kategorisinin** ortalamasından yüksekse getir."
    
- Burada "Ortalama" sabit bir sayı değildir (Tüm ürünlerin ortalaması 500 TL olabilir ama Elektronik ortalaması 10.000 TL'dir). Her satır için o satırın kategorisine bakıp yeniden hesaplama yapmak gerekir.
    

---

### Deep Dive: Performans Tuzağı ve N+1 Problemi

Bir Junior Developer olarak bu konu mülakatlarda senin "Veritabanı Motoru"nu (Engine) ne kadar iyi anladığını gösterir.

#### 1. Çalışma Mantığı (Satır Satır İşleme)

Standart Subquery ile arasındaki en büyük fark şudur:

- **Normal Subquery:** İçerideki sorgu **1 kere** çalışır, sonucu hafızaya alır, dışarıdaki sorgu bu sonucu kullanır.
    
- **Correlated Subquery:** Dış tablodaki (Outer Table) **her bir satır için** içerideki sorgu (Inner Query) yeniden çalışır.
    
    - Eğer dış tabloda 1 milyon satır varsa, iç sorgu 1 milyon kere çalışır! Buna yazılımda **N+1 Problemi** denir.
        

#### 2. Kod Anatomisi

İç sorgunun, dış sorguya "bağımlı" olduğunu `WHERE` bloğundaki bağlantıdan anlarsın.

SQL

```sql
SELECT p1.Name, p1.Price, p1.CategoryId
FROM Products p1
WHERE p1.Price > (
    SELECT AVG(p2.Price)
    FROM Products p2
    WHERE p2.CategoryId = p1.CategoryId -- KİLİT NOKTA: Dışarıdaki p1'i içeriye bağladık.
);
```

- Bu sorgu tek başına çalışmaz (Inner Query'i seçip run dersen hata verir, çünkü `p1` kim bilmiyor).
    

#### 3. EXISTS Operatörü ile Kullanımı - _Best Practice_

Correlated Subquery'nin en verimli kullanıldığı yer `EXISTS` komutudur.

- **Senaryo:** "Hiç sipariş vermemiş müşterileri bul."
    
- `NOT EXISTS` kullanıldığında, SQL motoru iç sorguda ilk eşleşmeyi bulduğu anda aramayı keser (Short-Circuit). Bu, `COUNT(*) > 0` saymaktan çok daha hızlıdır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Optimization (Window Functions):** Mülakatta _"Correlated Subquery çok yavaş çalışıyor, bunu nasıl hızlandırırsın?"_ diye sorarlar.
    
    - **Cevap:** "Eğer bir satırı grubuyla kıyaslıyorsam (Örn: Kategori ortalaması), Correlated Subquery yerine **Window Functions** (`AVG(Price) OVER(PARTITION BY CategoryId)`) kullanırım. Bu, N kere çalışmak yerine tek seferde hesaplar."
        
2. **Entity Framework Core Tuzağı:** Bazen yazdığın LINQ sorgusu arka planda Correlated Subquery üretir:
    
    C#
    
    ```csharp
    var result = context.Users
        .Where(u => u.Orders.Any()) // Her kullanıcı için sipariş tablosuna git-bak (EXISTS)
        .ToList();
    ```
    
    Bu genellikle `EXISTS` ürettiği için performanslıdır. Ancak karmaşık hesaplamalarda EF Core'un ürettiği SQL'i "SQL Profiler" ile izlemek gerekir.
    
3. **Toplu Update İşlemleri:** Bazen bir tablodaki veriyi, başka tablodaki güncel veriyle eşitlemek istersin. `UPDATE Products SET Price = (SELECT NewPrice FROM PriceList WHERE Id = Products.Id)` Bu zorunlu bir Correlated Subquery işlemidir ve büyük tablolarda sunucuyu yorar.
    

Correlated Subqueries, SQL'in en güçlü ama en maliyetli silahlarından biridir. Mantığını "Döngü" olarak kodlarsan asla unutmazsın.