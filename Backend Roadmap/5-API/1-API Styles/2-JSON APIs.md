JSON, verileri depolamak ve taşımak için kullanılan metin tabanlı (text-based), hafif bir formattır. Adında "JavaScript" geçse de, dillerden bağımsızdır (Language Agnostic). Yani C# ile oluşturduğun bir JSON'ı, Android (Kotlin) veya iOS (Swift) uygulaması sorunsuzca okur.

**Neden XML Değil de JSON?** Eskiden XML kullanılırdı (`<name>Ahmet</name>`). Ancak XML hantaldır, çok yer kaplar ve okuması zordur. JSON ise çok daha az karakter kullanır (`"name": "Ahmet"`), daha az bant genişliği (Bandwidth) harcar ve tarayıcılar (Browser) tarafından doğal olarak desteklenir.

### JSON'ın Anatomisi

JSON sadece iki temel yapıdan oluşur:

1. **Objects (Nesneler):** Süslü parantez `{}` içindedir. Anahtar-Değer (Key-Value) ikililerinden oluşur.
    
2. **Arrays (Diziler):** Köşeli parantez `[]` içindedir. Değer listesidir.
    

**Örnek Bir JSON Dosyas:**

JSON

```json
{
  "userId": 101,
  "fullName": "Ahmet Yılmaz",
  "isActive": true,
  "roles": ["Admin", "Editor"], // Array
  "address": {                   // Nested Object (İç içe nesne)
    "city": "İstanbul",
    "zipCode": 34000
  },
  "lastLogin": null
}
```

---

### C# Dünyasında JSON: Serialization & Deserialization

Bir Backend Developer olarak mülakatlarda veya günlük hayatta en çok duyacağın iki terim şunlardır:

1. **Serialization (Serileştirme):** C# nesnesini -> JSON string'ine çevirmektir. (API'den cevap dönerken yapılır).
    
2. **Deserialization (Tersine Serileştirme):** JSON string'ini -> C# nesnesine çevirmektir. (API'ye veri gelirken yapılır).
    

#### Kütüphaneler: `System.Text.Json` vs `Newtonsoft.Json`

- **Newtonsoft.Json (Json.NET):** Yıllarca sektörün kralıydı. Hala çok yaygındır ama yavaş yavaş tahtını bırakıyor.
    
- **System.Text.Json:** .NET Core 3.0 ile gelen, Microsoft'un yerleşik (Built-in) kütüphanesidir. Çok daha performanslıdır. Biz örneklerimizde bunu kullanacağız.
    

---

### Kod Örnekleri (C#)

Senaryomuz: Bir e-ticaret siparişini JSON'a çevirip göndermek ve gelen JSON'ı okumak.

#### 1. Model Hazırlığı

Önce C# class'ımızı tanımlayalım.

C#

```csharp
public class Order
{
    public int OrderId { get; set; }
    public string CustomerName { get; set; }
    public decimal TotalAmount { get; set; }
    public DateTime OrderDate { get; set; }
    public List<string> Items { get; set; }
}
```

#### 2. Serialization (C# -> JSON)

Veritabanından veriyi çektin, Frontend'e göndereceksin.

C#

```csharp
using System.Text.Json; // Kütüphaneyi ekle

public string ConvertOrderToJson()
{
    // C# Nesnesi oluştur
    var myOrder = new Order
    {
        OrderId = 55,
        CustomerName = "Mehmet Demir",
        TotalAmount = 150.50m,
        OrderDate = DateTime.Now,
        Items = new List<string> { "Laptop", "Mouse" }
    };

    // Serialize işlemi
    // WriteIndented = true -> JSON'ı okunabilir (güzel formatlı) yapar. Production'da false olması daha iyidir (boyut küçülür).
    var options = new JsonSerializerOptions { WriteIndented = true };
    string jsonString = JsonSerializer.Serialize(myOrder, options);

    return jsonString;
}
```

**Çıktı (JSON):** Dikkat et: C# özelliklerin `PascalCase` (OrderId) idi, ama JSON standartlarında genelde `camelCase` (orderId) tercih edilir. `System.Text.Json` varsayılan olarak olduğu gibi çevirir, ancak ayar yapılabilir.

JSON

```json
{
  "OrderId": 55,
  "CustomerName": "Mehmet Demir",
  "TotalAmount": 150.50,
  "OrderDate": "2026-01-15T10:00:00",
  "Items": [
    "Laptop",
    "Mouse"
  ]
}
```

#### 3. Deserialization (JSON -> C#)

Frontend sana bir sipariş gönderdi (POST request). Bunu C# nesnesine çevirip veritabanına kaydetmelisin.

C#

```csharp
public void ProcessJsonOrder(string jsonInput)
{
    // Gelen JSON verisi (Simülasyon)
    string json = @"{ ""OrderId"": 99, ""CustomerName"": ""Ayşe"", ""TotalAmount"": 200 }";

    // Deserialize işlemi
    // <Order> diyerek hangi kalıba dökeceğimizi söylüyoruz.
    Order newOrder = JsonSerializer.Deserialize<Order>(json);

    Console.WriteLine($"Sipariş ID: {newOrder.OrderId}");
    Console.WriteLine($"Müşteri: {newOrder.CustomerName}");
}
```

---

### Backend Developer İçin Kritik İpuçları

#### 1. İsimlendirme Standartları (Naming Policies)

JavaScript dünyası (Frontend) değişkenleri `camelCase` (userName) kullanır. C# dünyası `PascalCase` (UserName) kullanır. API yazarken bu uyumu sağlamak zorundasın.

- **Global Ayar:** `Program.cs` içinde API konfigürasyonu yaparken `PropertyNamingPolicy = JsonNamingPolicy.CamelCase` diyerek tüm API'nin otomatik dönüşüm yapmasını sağlarsın.
    
- **Özel Ayar (Attribute):** Sadece belirli bir alanı değiştirmek istersen:
    

C#

```csharp
using System.Text.Json.Serialization;

public class User
{
    // C# tarafında 'FirstName' ama JSON'a çevrilirken 'first_name' olsun.
    [JsonPropertyName("first_name")]
    public string FirstName { get; set; }
    
    // Bu alanı JSON'a hiç dahil etme (Örn: Şifre alanı)
    [JsonIgnore]
    public string Password { get; set; }
}
```

#### 2. Tarih Formatı (Date Handling)

JSON standardında özel bir "Date" tipi yoktur. Tarihler String olarak taşınır.

- Standart format **ISO 8601**'dir: `YYYY-MM-DDTHH:mm:ss` (Örn: `2026-01-15T14:30:00`).
    
- Backend geliştirici olarak tarihleri hep bu formatta gönderip alman, saat dilimi (Timezone) sorunlarını minimize eder.
    

#### 3. Döngüsel Başvuru Hatası (Circular Reference)

Entity Framework kullanırken N+1 probleminde gördüğümüz ilişkiler (Yazar -> Kitaplar -> Yazar -> Kitaplar...) JSON'a çevrilirken sonsuz döngüye girer ve uygulama patlar (`JsonException: A possible object cycle was detected`).

- **Çözüm:** `ReferenceHandler.IgnoreCycles` ayarını kullanmak veya DTO (Data Transfer Object) kullanarak sadece gerekli alanları maplemektir.