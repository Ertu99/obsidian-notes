

Yazılım dünyasının en büyük "aşk ve nefret" ilişkisi buradadır. ORM, geliştiricinin hayatını kurtarır ama yanlış kullanılırsa performansın katili olur. Bu konuyu sadece "Entity Framework kullanmak" olarak değil, **"Impedance Mismatch" (Empedans Uyuşmazlığı)** problemi ve mimari trade-off'lar üzerinden mühendislik derinliğinde ele alacağız.

---

### 1. ORM Nedir? (Felsefe: Çevirmen)

Veritabanları İlişkisel (Relational) mantıkla çalışır (Tablolar, Satırlar, Foreign Key'ler).

C# ise Nesne Yönelimli (Object Oriented) mantıkla çalışır (Class'lar, Inheritance, List'ler).

Bu iki dünya birbirine yabancıdır.

- SQL'de `List<Siparis>` diye bir şey yoktur, `Siparisler` tablosunda satırlar vardır.
    
- C#'ta `JOIN` diye bir komut yoktur, `user.Orders` diye referanslar vardır.
    

**ORM**, bu iki dünya arasındaki **Çevirmen (Translator)**'dir. Sen C# ile `context.Users.Add(user)` dersin, ORM bunu arka planda `INSERT INTO Users...` SQL komutuna çevirir.

---

### 2. Impedance Mismatch (Uyuşmazlık Problemi)

Mülakatlarda sorulan "Neden ORM kullanırken dikkatli olmalıyız?" sorusunun cevabı budur. Nesne dünyası ile Veri dünyası birbirine tam oturmaz.

1. **Inheritance (Kalıtım):** C#'ta `Mudur : Personel` yapabilirsin. Ama SQL'de "Tablo Kalıtımı" diye standart bir yapı yoktur. ORM bunu çözmek için TPH (Table Per Hierarchy) gibi taklalar atar.
    
2. **Identity (Kimlik):** C#'ta iki nesne aynı bellekteyse (`Ref A == Ref B`) eşittir. Veritabanında ise Primary Key'leri aynıysa (`ID = 5`) eşittir.
    
3. **Encapsulation:** Veritabanında veriler açıktır. C#'ta ise `private` alanlar vardır. ORM, reflection ile bu sınırları aşmak zorundadır.
    

---

### 3. ORM Türleri: Full ORM vs Micro ORM

Bir .NET geliştiricisi olarak iki ana aracı bilmelisin.

#### A. Full ORM (Örn: Entity Framework Core, Hibernate)

"Her şeyi ben yaparım" diyen araçtır.

- **Özellik:** SQL yazmazsın. LINQ kullanırsın (`db.Users.Where(u => u.Age > 18)`).
    
- **Artısı:** Geliştirme hızı çok yüksektir (Productivity). Change Tracking (Değişiklik takibi) yapar.
    
- **Eksisi:** Performans kaybı olabilir (Overhead). Bazen çok karmaşık SQL üretir.
    

#### B. Micro ORM (Örn: Dapper)

"Ben sadece veriyi nesneye çeviririm, SQL'i sen yaz" diyen araçtır.

- **Özellik:** SQL sorgusunu elle yazarsın (`SELECT * FROM Users...`), Dapper sonucu `User` nesnesine doldurur (Mapping).
    
- **Artısı:** Işık hızındadır (Performans). SQL'e tam hakimiyet sağlar.
    
- **Eksisi:** CRUD işlemlerini (Insert, Update) elle yazmak zaman alır.
    

---

### 4. Code First vs Database First

ORM kullanırken projeye başlama stratejisi önemlidir.

1. **Code First (Modern):** Önce C# class'larını (`User.cs`) yazarsın. ORM, bu class'lara bakarak veritabanını (`Users` tablosunu) sıfırdan yaratır (Migration).
    
2. **Database First (Geleneksel):** Önce SQL'de tabloları tasarlarsın. ORM, bu tablolara bakarak C# class'larını otomatik üretir (Scaffolding).
    

---

### 5. ORM'in Baş Belası: N+1 Problemi

ORM kullanan her junior'ın (ve bazen senior'ın) düştüğü performans tuzağıdır.

**Senaryo:** Blog yazılarını ve her yazının yazarını listelemek istiyorsun.

C#

```csharp
// 1. Sorgu: Tüm blogları çek (SELECT * FROM Blogs)
var blogs = context.Blogs.ToList(); 

foreach (var blog in blogs) 
{
    // HATA! Her döngüde Yazar için veritabanına bir daha gider.
    // N tane blog varsa, N tane ekstra sorgu atılır.
    // Toplam Sorgu: 1 (Bloglar) + N (Yazarlar)
    Console.WriteLine(blog.Author.Name); 
}
```

Eğer 1000 blog varsa, veritabanına 1001 kere gidilir. Sistem kilitlenir.

Çözüm (Eager Loading):

ORM'e en başta "Yazarları da getir" demelisin (JOIN yap).

C#

```csharp
// Tek sorguda (JOIN ile) hem blogları hem yazarları getirir.
var blogs = context.Blogs.Include(b => b.Author).ToList(); 
```

---

### 6. ORM'in Faydaları ve Zararları

kaynağına göre özetleyelim:

- **Pros (Avantajlar):**
    
    - **Verimlilik:** Daha az kod yazarsın (Boilerplate azalır).
        
    - **Soyutlama:** Veritabanı türünü (SQL Server'dan PostgreSQL'e) değiştirmek kolaydır, çünkü sen SQL değil C# yazıyorsun.
        
    - **Bakım:** Nesne modeli ile çalışmak daha temizdir.
        
- **Cons (Dezavantajlar):**
    
    - **Performans:** Ham SQL her zaman daha hızlıdır.
        
    - **Öğrenme Eğrisi:** EF Core'un detaylarını öğrenmek, SQL öğrenmekten zor olabilir.
        
    - **Kontrol Kaybı:** Arka planda ne tür bir SQL oluştuğunu kontrol edemeyebilirsin (bazen "Canavar Sorgular" oluşur).
        

---

