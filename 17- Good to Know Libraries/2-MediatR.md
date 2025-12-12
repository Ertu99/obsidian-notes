
Şimdi .NET dünyasında **Clean Architecture** ve **CQRS** denince akla gelen ilk kütüphaneye, kodun "Trafik Polisi"ne, **MediatR**'a geliyoruz.

Çoğu geliştirici MediatR'ı sadece "Controller'ı temizlemek için bir araç" sanır.

Oysa MediatR; Cross-Cutting Concerns (Pipeline Behaviors), Vertical Slice Architecture (Dikey Mimari) ve In-Process Messaging stratejisinin kalbidir.

Bu konuyu; **Pattern (Mediator)**, **CQRS ile İlişkisi**, **Pipeline (Boru Hattı) Mimarisi** ve **Notification (Pub/Sub)** yetenekleri üzerinden, bir Mimar derinliğinde, tek seferde ve eksiksiz inceleyelim.

---

### 1. Felsefe: Garson Analojisi (Neden İhtiyacımız Var?)

MediatR olmadan klasik katmanlı mimaride Controller şuna benzer:

C#

```cs
public class OrderController
{
    // Constructor Cehennemi (Constructor Injection Hell)
    public OrderController(
        IOrderService orderService, 
        IEmailService emailService, 
        ILogger logger, 
        IMapper mapper, 
        IValidator validator) { ... }

    public void Create() { ... }
}
```

Controller, işi yapmak için herkese bağımlıdır. Bir servis değişse Controller da değişir (**Tight Coupling**).

MediatR ile:

Controller bir müşteridir. Mutfaktaki şefleri (Servisleri) tanımaz. Sadece Garsonu (Mediator) tanır.

1. **Müşteri (Controller):** "Ben 'Hamburger' istiyorum (Command)." der ve Garsona verir.
    
2. **Garson (Mediator):** Mutfağa gider, "Hamburger" yapabilen şefi (Handler) bulur ve siparişi ona verir.
    
3. **Şef (Handler):** Yemeği yapar.
    

**Sonuç:** Controller'ın Constructor'ında sadece `IMediator` vardır. Bağımlılıklar sıfıra iner.

---

### 2. Mimari: Request ve Handler (CQRS Temeli)

MediatR iki temel bileşen üzerine kuruludur.

1. **Request (İstek/Mesaj):** Sadece veri taşıyan basit bir sınıftır (DTO). Davranış içermez.
    
    - `IRequest<TResponse>`: Cevap beklenen istek (Command/Query).
        
    - `CreateOrderCommand` veya `GetProductQuery`.
        
2. **Handler (İşleyici):** İşi yapan sınıftır.
    
    - `IRequestHandler<TRequest, TResponse>`: İsteği alır, işler, cevabı döner.
        

C#

```cs
// 1. Mesaj (Command)
public record CreateUserCommand(string Name, string Email) : IRequest<int>;

// 2. İşleyici (Handler)
public class CreateUserHandler : IRequestHandler<CreateUserCommand, int>
{
    private readonly AppDbContext _db; // Bağımlılıklar burada!
    public CreateUserHandler(AppDbContext db) => _db = db;

    public async Task<int> Handle(CreateUserCommand request, CancellationToken ct)
    {
        var user = new User { Name = request.Name };
        _db.Users.Add(user);
        await _db.SaveChangesAsync(ct);
        return user.Id;
    }
}

// 3. Kullanım (Controller)
var userId = await _mediator.Send(new CreateUserCommand("Ahmet", "a@b.com"));
```

---

### 3. Mühendislik Harikası: Pipeline Behaviors (AOP)

MediatR'ın en güçlü özelliği budur. **Aspect Oriented Programming (AOP)** yapmanı sağlar.

Sorun: Her Handler'ın içine try-catch, logger.Log, validator.Validate yazmak kod tekrarıdır.

Çözüm: Pipeline Behavior.

Handler çalışmadan önce ve sonra araya giren middleware'lerdir.

Örnek: Validation Behavior (FluentValidation Entegrasyonu)

Her işlemden önce otomatik validasyon yapar.

C#

```cs
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators) 
        => _validators = validators;

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        // 1. ÖNCE: Validasyonları çalıştır
        var context = new ValidationContext<TRequest>(request);
        var failures = _validators
            .Select(v => v.Validate(context))
            .SelectMany(result => result.Errors)
            .Where(f => f != null)
            .ToList();

        if (failures.Count != 0) throw new ValidationException(failures);

        // 2. SONRA: Asıl Handler'a git
        return await next();
    }
}
```

Bunu `Program.cs`'e kaydettiğinde, artık **hiçbir Handler'ın içine if-else ile validasyon yazmazsın.** Sistem otomatik yapar.

---

### 4. Notifications (In-Process Pub/Sub)

_mediator.Send() birebir iletişimdir (1 İstek -> 1 Handler).

Peki ya "Kullanıcı Kaydoldu" olayını hem "Email Servisi", hem "Log Servisi", hem de "Cache Servisi" duymak istiyorsa?

Burada **`INotification`** kullanılır.

- **Gönderim:** `await _mediator.Publish(new UserCreatedNotification(userId));`
    
- **Dinleyiciler:** İstediğin kadar `INotificationHandler<UserCreatedNotification>` yazabilirsin.
    
- MediatR hepsini tek tek bulur ve çalıştırır.
    

**Mühendislik Riski:** Bu işlem varsayılan olarak **Senkron ve Sıralı** çalışır (biri hata verirse diğerleri çalışmayabilir veya hepsi birbirini bekler). Gerçekten asenkron ve güvenilir bir yapı için **MassTransit/RabbitMQ** kullanmak gerekir. MediatR sadece bellek içi (In-Memory) basit işler içindir.

---

### 5. Mimari Desen: Vertical Slice Architecture

MediatR kullanan projelerde klasör yapısı değişir.

- **Eski (Layered):** `Services`, `Repositories`, `DTOs` klasörleri. (İlgili kodlar dağınık).
    
- **Yeni (Vertical):** `Features` klasörü.
    
    - `Features/Users/CreateUser/` klasörünün içinde:
        
        - `CreateUserCommand.cs`
            
        - `CreateUserHandler.cs`
            
        - `CreateUserValidator.cs`
            
        - `CreateUserDto.cs`
            
    - **Fayda:** Bir özellikle ilgili her şey yan yanadır. Bir kodu değiştirirken projede kaybolmazsın.
        

---

### 6. Performans ve Reflection Maliyeti

MediatR, doğru Handler'ı bulmak için çalışma zamanında **Service Locator** ve **Reflection** kullanır.1

- **Soru:** "Bu performans kaybı yaratır mı?"2
    
- **Cevap:** Teorik olarak evet, doğrudan metod çağırmaktan yavaştır. Ancak bu kayıp **nanosaniyeler** mertebesindedir. Veritabanı çağrısı (milisaniyeler) yanında ihmal e3dilebilir. Sağladığı temiz mimari avantajı, bu maliyete değer.
    

---
