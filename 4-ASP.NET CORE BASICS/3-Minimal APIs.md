
"MVC varken buna ne gerek var?" diye düşünebilirsin. Haklısın da. Ama modern yazılım dünyası (Microservices, Cloud Native, Serverless) değiştikçe, .NET'in o eski ağır yapısı (Boilerplate code) hantal kalmaya başladı. Node.js (Express) veya Go (Gin) ile yazanlar "C# çok karmaşık, bir endpoint açmak için 10 satır kod yazıyorum" diyordu.

Microsoft buna **Minimal API** ile cevap verdi. Bu konuyu sadece "kısa kod yazmak" olarak değil, **performans ve mimari** açısından Controller yapısıyla kıyaslayarak inceleyeceğiz.

---

### 1. Minimal API Nedir? (Felsefe: Less Ceremony)

Geleneksel MVC/Web API yaklaşımında bir "Merhaba Dünya" demek için bile şunlara ihtiyacın vardır:

1. Bir `Controllers` klasörü.
    
2. `ControllerBase`den türeyen bir sınıf.
    
3. `[ApiController]`, `[Route]` attribute'ları.
    
4. Startup dosyasında `AddControllers()` ve `MapControllers()` ayarları.
    

Buna **"Ceremony" (Tören)** denir. İş yapmak için çok fazla hazırlık gerekir.

**Minimal API**, bu töreni çöpe atar. C#'ın **Top-Level Statements** özelliğini kullanarak, uygulamanın giriş dosyası olan `Program.cs` içinde doğrudan endpoint tanımlamanı sağlar.

---

### 2. Controller vs Minimal API (Kod Karşılaştırması)

Mühendislik farkını görmek için aynı işi yapan iki koda bakalım.

**Eski Usül (Controller API):**

C#

```csharp
// Dosya: WeatherController.cs
[ApiController]
[Route("[controller]")]
public class WeatherController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        return ...; // Veriyi dön
    }
}
```

**Yeni Usül (Minimal API):**

C#

```csharp
// Dosya: Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Tek satırda tanımlama
app.MapGet("/weather", () => 
{
    return ...; // Veriyi dön
});

app.Run();
```

Farkı görüyor musun? Class yok, namespace yok, inheritance (kalıtım) yok. Sadece **Route** ve **Handler (Lambda Expression)** var.

---

### 3. Kaputun Altı: Neden Daha Performanslı?

Minimal API'lar sadece "daha az kod" demek değildir; aynı zamanda **daha az işlem yükü** demektir.

Controller yapısında bir istek geldiğinde (Request Pipeline):

1. Routing eşleşir.
    
2. **Controller Factory** devreye girer (Reflection ile Controller sınıfını bulur).
    
3. Controller'ın örneğini (Instance) oluşturur.
    
4. **Filter Pipeline** çalışır (Authorization Filter, Action Filter, Exception Filter...).
    
5. Action (Metot) çalıştırılır.
    

**Minimal API'da ise:**

1. Routing eşleşir.
    
2. Doğrudan Lambda fonksiyonu (Handler) çalışır.
    

Aradaki o karmaşık "Controller oluşturma, Filter pipeline'dan geçme" süreçleri yoktur. Bu yüzden **başlangıç hızı (Cold Start)** ve **RAM kullanımı** açısından Minimal API'lar daha verimlidir. Özellikle AWS Lambda veya Azure Functions gibi "Serverless" ortamlarda bu hız farkı paraya dönüşür.

---

### 4. Dependency Injection (DI) Nasıl Çalışır?

Controller'da Constructor Injection kullanıyorduk. Minimal API'da Constructor yok. Peki servislere nasıl erişeceğiz?

Cevap: **Metot Enjeksiyonu (Method Injection).**

C#

```csharp
// Lambda'nın parametresine servisi yazman yeterli
app.MapGet("/users/{id}", (int id, IUserService userService) => 
{
    // Framework, IUserService'i container'dan bulup buraya verir.
    return userService.GetById(id);
});
```

Framework o kadar akıllıdır ki, hangisinin URL parametresi (`id`), hangisinin Servis (`userService`) olduğunu tipinden anlar.

---

### 5. Mimari Risk: Spagetti Kod Tehlikesi

Minimal API harika ama büyük bir tehlikesi var: **Program.cs Şişmesi.**

Eğer 100 tane endpoint'i alt alta Program.cs içine yazarsan, o dosya 5000 satır olur ve yönetilemez hale gelir.

Bir mühendis olarak burada Extension Methods veya Modüler Yapı kurmalısın.

**Best Practice (Endpoint Gruplama):**

C#

```csharp
// Program.cs temiz kalır
app.MapUserEndpoints();

// Ayrı bir dosya: UserEndpoints.cs
public static class UserEndpoints
{
    public static void MapUserEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/users");

        group.MapGet("/", GetUsers);
        group.MapPost("/", CreateUser);
    }
    
    // İş mantığı burada veya Service katmanında
    static async Task<IResult> GetUsers(IUserService service) { ... }
}
```

---

### 6. Karar Anı: Ne Zaman Hangisi?

Mülakatlarda "Projende neden Minimal API seçtin?" diye sorarlar. İşte cevabın:

| **Özellik**        | **Minimal API**                                | **Controller-based API**                         |
| ------------------ | ---------------------------------------------- | ------------------------------------------------ |
| **Kullanım Alanı** | Microservices, Serverless, Küçük-Orta Projeler | Büyük Kurumsal Monolitler (Enterprise Monoliths) |
| **Öğrenme Eğrisi** | Düşük (Yeni başlayanlar için kolay)            | Orta (MVC desenini bilmek gerekir)               |
| **Performans**     | Daha Yüksek (Daha az katman)                   | Standart                                         |
| **Organizasyon**   | Manuel yapmalısın (Disiplin gerektirir)        | Klasör yapısı hazırdır (Disiplin dayatır)        |
| **Features**       | Çoğu özellik var (Validation, Auth) ama manuel | Built-in Filters, Model Binding çok güçlü        |

---

