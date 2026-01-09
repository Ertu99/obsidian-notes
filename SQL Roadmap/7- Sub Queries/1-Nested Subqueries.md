Nested Subquery, bir "Matruşka Bebeği" veya "Inception" filmi gibidir: Sorgu içinde sorgu, onun da içinde bir başka sorgu vardır.

**Neden Kullanılır?** Bazen bir veriye ulaşmak için birden fazla adımı takip etmen gerekir.

- _Adım 1:_ En pahalı ürünü bul.
    
- _Adım 2:_ Bu ürünü sipariş eden sipariş numaralarını bul.
    
- _Adım 3:_ Bu siparişleri veren müşterilerin isimlerini getir. Bu zincirleme mantığı tek bir SQL komutunda yazmak için iç içe sorgular kullanılır.
    

---

### Deep Dive: Rüya İçinde Rüya Mantığı

Bir Junior Developer olarak iç içe sorgular ilk bakışta korkutucu görünebilir. Ancak sırrı şudur: **Okumaya ve çalıştırmaya her zaman en içteki parantezden başlanır.**

#### 1. Çalışma Sırası (Inside-Out)

SQL Motoru bu karmaşık yapıyı şu sırayla çözer:

1. **Innermost (En İçteki):** Önce en içteki sorgu çalışır ve bir sonuç (genellikle tek bir değer veya bir ID listesi) üretir.
    
2. **Intermediate (Ortadaki):** En içten gelen sonucu alır, kendi işini yapar ve dışarıya yeni bir sonuç verir.
    
3. **Outermost (En Dıştaki):** Ortadan gelen sonucu alır ve final tabloyu oluşturur.
    

#### 2. Örnek Senaryo

"Fiyatı 50.000 TL üzeri olan ürünleri alan müşterilerin adını getir."

SQL

```sql
SELECT Name FROM Users WHERE Id IN (                -- 3. ADIM (Dış): Müşterileri bul
    SELECT UserId FROM Orders WHERE ProductId IN (  -- 2. ADIM (Orta): Siparişleri bul
        SELECT Id FROM Products WHERE Price > 50000 -- 1. ADIM (İç): Ürünleri bul
    )
);
```

#### 3. Okunabilirlik Sınırı (The Readability Trap)

Prompt'ta "karmaşıklaşabilir" dendiği gibi, 3 katmandan sonrası kodu okuyan kişi için (ve 6 ay sonraki senin için) bir işkencedir.

- **Tavsiye:** Eğer 2-3 katmanı geçiyorsan, bu yapıyı **JOIN** kullanarak veya daha modern bir yöntem olan **CTE (Common Table Expressions)** ile yazmak "Clean Code" açısından daha doğrudur.
    

---

### Backend Developer İçin Neden Önemli?

1. **LINQ Zincirleri:** Backend tarafında C# ile şöyle bir kod yazdığında:
    
    C#
    
    ```csharp
    var result = context.Users
        .Where(u => u.Orders.Any(o => o.Product.Price > 50000))
        .ToList();
    ```
    
    Entity Framework Core, arka planda bunu optimize eder ancak bazen yukarıdaki gibi `EXISTS` veya `IN` kullanan Nested Subquery'ler oluşturur. Bu SQL çıktısını okuyabilmek, performans sorunlarını çözmek için şarttır.
    
2. **Veri Temizliği (DELETE/UPDATE):** Özellikle veri silerken çok işe yarar.
    
    - _"Hiç siparişi olmayan ve son 1 yıldır giriş yapmamış kullanıcıları sil"_ dediğinde, kimi sileceğini bulmak için mecburen `DELETE FROM Users WHERE Id IN (SELECT ...)` şeklinde iç içe sorgu yazarsın.
        
3. **Performans Maliyeti:** Nested Subquery'ler bazen veritabanı motoru tarafından otomatik olarak JOIN'e dönüştürülüp optimize edilir (Query Optimizer). Ancak kötü yazılmış, indeks kullanmayan iç içe sorgular, her satır için tekrar çalışarak (Correlated) sistemi kilitleyebilir.
    

Nested Subqueries konusu, karmaşık mantıkları kurabilmen için güçlü bir araçtır. Ancak "Çalışıyorsa dokunma" değil, "Daha okunabilir yazılabilir mi?" diye sorgulaman gereken bir alandır.