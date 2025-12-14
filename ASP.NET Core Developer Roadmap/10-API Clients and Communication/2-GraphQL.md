REST mimarisinin "kaynak" (resource) odaklı yapısını ve sınırlarını (Over-fetching/Under-fetching) konuştuk. Şimdi ise Facebook'un bu sorunlara kızıp geliştirdiği, API dünyasının "Game Changer" teknolojisine, **GraphQL**'e geçiyoruz.

REST, sunucunun patron olduğu ("Ben sana ne verirsem onu alırsın") bir yapıdır.

GraphQL ise istemcinin (Client) patron olduğu ("Bana tam olarak şunları ver, fazlasını istemem") bir yapıdır.

Bu konuyu sadece "esnek sorgu" olarak değil; **Single Endpoint (Tek Uç Nokta)** mimarisi, **N+1 Problemi (DataLoader Çözümü)** ve **Security (Complexity Analysis)** gibi derin mühendislik katmanlarıyla, tek seferde ve eksiksiz inceleyeceğiz.

---

### 1. Felsefe: REST'in Ölümcül Sorunu (Over & Under Fetching)

GraphQL'i anlamak için önce REST'in neden yetmediğini anlamalıyız.

**Senaryo:** Bir blog sayfasında yazarın adını ve son 3 yazısının başlığını göstereceksin.

- **REST Yaklaşımı:**
    
    1. `GET /authors/1` -> Yazarın tüm bilgisini (Ad, Soyad, Adres, TC, Telefon...) döner. **(Over-fetching: Gereksiz veri)**.
        
    2. `GET /authors/1/posts` -> Yazarın 100 tane yazısını döner. Sen sadece 3 başlık istiyordun ama içerik, tarih, yorumlar hepsi geldi. **(Over-fetching)**.
        
    3. Ya da bu iki veriyi almak için iki kere sunucuya gitmen gerekir. **(Under-fetching / Multiple Roundtrips)**.
        
- GraphQL Yaklaşımı:
    
    Tek bir istek atarsın:
    
    GraphQL
    
    ```
    query {
      author(id: 1) {
        name
        posts(last: 3) {
          title
        }
      }
    }
    ```
    
    **Sonuç:** Sunucu sadece istenen alanları içeren, tertemiz bir JSON döner. Tek seferde iş biter.
    

---

### 2. Mimari: Schema, Type System ve SDL

GraphQL bir veritabanı değildir. Bir **Sorgu Dili (Query Language)** ve bir **Runtime**'dır. Arkada SQL, NoSQL veya başka bir REST API olabilir.

Her şey bir **Şema (Schema)** ile başlar. Bu, API'nin "Menüsü"dür.

GraphQL

```
# SDL (Schema Definition Language)

type Author {
  id: ID!
  name: String!
  posts: [Post] # İlişki
}

type Post {
  id: ID!
  title: String!
  content: String
}

type Query { # Giriş Noktaları
  getAuthor(id: ID): Author
  getAllPosts: [Post]
}
```

**Mühendislik Notu (Strongly Typed):** GraphQL "Tip Güvenli" (Strongly Typed) bir yapıdır. Şemada olmayan bir alanı istersen, sunucu daha kodu çalıştırmadan "Validasyon Hatası" verir. REST'teki gibi "Acaba null mu geldi, string mi geldi?" stresi yoktur.

---

### 3. Operasyon Türleri (Query, Mutation, Subscription)

REST'te GET, POST, PUT, DELETE vardı. GraphQL'de ise 3 temel operasyon vardır:

1. **Query (Sorgu):** Veri okumak içindir (REST GET karşılığı).
    
2. **Mutation (Değişim):** Veri değiştirmek içindir (REST POST/PUT/DELETE karşılığı).
    
    - _Önemli:_ Mutation sonucunda, değişen verinin son halini de aynı anda isteyebilirsin.
        
3. **Subscription (Abonelik):** **Real-Time (Gerçek Zamanlı)** iletişim içindir.
    
    - WebSocket üzerinden çalışır.
        
    - _Senaryo:_ "Yeni bir yorum gelirse beni uyar."
        

---

### 4. .NET Ekosistemi: Hot Chocolate

Attığın metinde GraphQL.NET ve Hot Chocolate geçmiş.

Bir .NET Mimarı olarak bilmelisin ki: Sektör standardı Hot Chocolate'tır. (Microsoft bile bunu önerir).

Performansı çok yüksektir ve **Code-First** yaklaşımını destekler. Yani sen C# class'larını yazarsın, o otomatik olarak GraphQL şemasını üretir.

C#

```cs
public class Query
{
    // Resolver Metodu
    public Author GetAuthor([Service] IAuthorRepository repo, int id) => repo.GetById(id);
}
```

---

### 5. Mühendislik Kabusu: N+1 Problemi

GraphQL'in (ve ORM'lerin) en büyük performans tuzağıdır.

**Senaryo:** 10 Yazar ve her yazarın kitaplarını istedin.

GraphQL

```
query {
  authors {     # 1. Sorgu: SELECT * FROM Authors (10 satır döndü)
    name
    books {     # 2. Sorun: Her yazar için TEK TEK veritabanına gider.
      title
    }
  }
}
```

Sonuç: 1 (Yazarlar) + 10 (Kitaplar) = 11 Veritabanı Sorgusu.

Eğer 1000 yazar çekersen, 1001 sorgu atılır. Sunucu kilitlenir.

Çözüm: DataLoader (Batching)

Facebook bu sorunu çözmek için DataLoader desenini geliştirdi.

- **Mantık:** İstekler hemen veritabanına gitmez. DataLoader onları "biriktirir" (örneğin 10ms bekler).
    
- **Toplu İş:** "Bana 1, 2, 3... 10 ID'li yazarların kitapları lazım" der.
    
- **Tek Sorgu:** `SELECT * FROM Books WHERE AuthorId IN (1, 2, 3... 10)`
    
- **Sonuç:** 1 (Yazarlar) + 1 (Toplu Kitaplar) = **2 Sorgu.**
    

---

### 6. Güvenlik: Complexity & Depth Limiting

REST'te endpoint bellidir. GraphQL'de ise istemci istediği kadar derine inebilir. Kötü niyetli biri (Hacker) şöyle bir sorgu gönderebilir:

GraphQL

```
query {
  author {
    posts {
      author {
        posts {
          author {
             ... # Sonsuz döngü (Recursive)
          }
        }
      }
    }
  }
}
```

Bu sorgu sunucunun RAM'ini bitirir (**DoS Attack**).

**Mühendislik Çözümü:**

1. **Max Depth Limiting:** "En fazla 3 katman derine inebilirsin" kuralı koymak.
    
2. **Cost Analysis (Maliyet Analizi):** Her alana puan verirsin. "Author: 1 puan, Posts: 5 puan". Sorgunun toplam puanı 100'ü geçerse reddedersin.
    
3. **Persisted Queries:** Sadece sunucuda kayıtlı (hashlenmiş) sorgulara izin verirsin. Dışarıdan rastgele sorgu gönderilemez.
    

---

### 7. REST vs GraphQL: Ne Zaman Hangisi?

Mülakat sorusu: "Madem GraphQL harika, REST öldü mü?"

Hayır.

|**Özellik**|**REST**|**GraphQL**|
|---|---|---|
|**Caching**|Çok Kolay (HTTP Caching standarttır)|Zor (Çünkü tek endpoint POST üzerinden çalışır)|
|**Öğrenme Eğrisi**|Düşük|Yüksek (Schema, Resolver, DataLoader...)|
|**Dosya Upload**|Standart|Karmaşık (Ekstra kütüphane gerekir)|
|**Bandwidth**|Yüksek (Gereksiz veri)|Düşük (Sadece gereken veri)|
|**Esneklik**|Düşük (Backend ne verirse o)|Yüksek (Frontend ne isterse o)|
|**Kullanım**|Public API, Microservices arası iletişim|Mobil App, Dashboard, Complex Frontend|

---
