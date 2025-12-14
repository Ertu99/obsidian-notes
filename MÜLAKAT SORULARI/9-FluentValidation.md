- **Amaç:** API’ye gelen **DTO**’ların (request modellerinin) **kurallara uygun** olup olmadığını kontrol etmek.
- **Tarz:** Attribute yerine **kodla kural** yazarsın (fluently).
- **Kapsam:** Basit kontrollerden (boş olmasın, uzunluk) karmaşık kurallara (cross-field, koşullu, koleksiyon, custom & async) kadar.

# Neden FluentValidation?

- **Okunabilirlik:** Kurallar tek yerde, akıcı cümle gibi: `RuleFor(x => x.Name).NotEmpty().Length(3, 100);`
- **Test edilebilirlik:** Validator’lar izole sınıflar; unit test çok kolay.
- **Esneklik:** Koşullu/kompleks kurallar, koleksiyonlar, nested nesneler, asenkron kontroller.
- **Katman temizliği:** Doğrulama **DTO** katmanında; entity’leri kirletmez (Data Annotations gibi attribute yığılmaz).

# Nasıl kurulur & bağlanır? ([ASP.NET](http://ASP.NET) Core)

> Paket: FluentValidation.AspNetCore

**Program.cs**

```csharp
using FluentValidation;
using FluentValidation.AspNetCore;

builder.Services
    .AddControllers();

// Otomatik server-side doğrulama için:
builder.Services
    .AddFluentValidationAutoValidation()
    .AddFluentValidationClientsideAdapters();

// Bu assembly’deki tüm validator’ları kaydet:
builder.Services.AddValidatorsFromAssemblyContaining<CreateHotelDtoValidator>();

```

> AddFluentValidationAutoValidation() sayesinde, [ApiController] aktifse, model binding ardından validator’lar otomatik çalışır; hatalıysa 400 Bad Request ve field bazlı hata listesi döner. Controller metoduna girmeden kestirir.

### Tipik 400 cevabı (ValidationProblemDetails)

```json
{
  "type": "<https://tools.ietf.org/html/rfc9110#section-15.5.1>",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": [ "Name is required", "Name length must be between 3 and 100." ],
    "Star": [ "Star must be between 1 and 5." ]
  }
}

```

# Temel kullanım (Hotel senaryosu)

## 1) DTO’lar

```csharp
public class CreateHotelDto
{
    public string Name { get; set; } = null!;
    public string City { get; set; } = null!;
    public int Star { get; set; }
}

public class UpdateHotelDto
{
    public string Name { get; set; } = null!;
    public string City { get; set; } = null!;
    public int Star { get; set; }
}

```

## 2) Validator sınıfları

```csharp
using FluentValidation;

public class CreateHotelDtoValidator : AbstractValidator<CreateHotelDto>
{
    public CreateHotelDtoValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Otel adı boş olamaz.")
            .Length(3, 100).WithMessage("Otel adı 3-100 karakter olmalı.");

        RuleFor(x => x.City)
            .NotEmpty().WithMessage("Şehir zorunlu.")
            .MaximumLength(100);

        RuleFor(x => x.Star)
            .InclusiveBetween(1, 5).WithMessage("Yıldız 1 ile 5 arasında olmalı.");
    }
}

public class UpdateHotelDtoValidator : AbstractValidator<UpdateHotelDto>
{
    public UpdateHotelDtoValidator()
    {
        Include(new CreateHotelDtoValidator()); // aynı kuralları paylaş

        // Update'e özgü ek kural örneği:
        // RuleFor(x => x.Name).Must(BeNonDuplicateNameForUpdate).WithMessage("...");
    }
}

```

> Include(...) ile tekrarı azaltabilirsin. WithMessage(...) ile özelleştirilmiş mesajlar ver.

# Orta/ileri seviye kurallar (özet)

- **Koşullu**:
    
    ```csharp
    RuleFor(x => x.City)
        .NotEmpty()
        .When(x => x.Star >= 4); // 4-5 yıldızda şehir zorunlu olsun
    
    ```
    
- **Özel (custom)**:
    
    ```csharp
    RuleFor(x => x.Name).Must(name => !name.Contains("test", StringComparison.OrdinalIgnoreCase))
        .WithMessage("İsim 'test' içeremez.");
    
    ```
    
- **Asenkron (DB kontrolü, benzersizlik)**:
    
    ```csharp
    RuleFor(x => x.Name).MustAsync(async (name, ct) =>
    {
        // ör: repository ile "var mı" kontrolü
        return !await hotelRepo.ExistsByNameAsync(name, ct);
    }).WithMessage("Bu isimde otel zaten var.");
    
    ```
    
- **Koleksiyon / RuleForEach**:
    
    ```csharp
    RuleForEach(x => x.Rooms).SetValidator(new RoomDtoValidator());
    
    ```
    
- **Cross-field**:
    
    ```csharp
    RuleFor(x => x)
        .Must(x => x.City != "İstanbul" || x.Star >= 4)
        .WithMessage("İstanbul için en az 4 yıldız gerekli.");
    
    ```
    
- **RuleSets** (farklı senaryolara farklı kural grupları):
    
    ```csharp
    RuleSet("Create", () => { /* create kuralları */ });
    RuleSet("Update", () => { /* update kuralları */ });
    // Validator’a özel çağırarak çalıştırabilirsin
    
    ```
    

# Çalışma akışı (pipeline)

1. **Model binding**: JSON → DTO.
2. **FluentValidation**: DTO doğrulanır.
3. Hatalıysa: **400** + hata listesi (controller’a girmez).
4. Geçerliyse: Controller/Service çalışır.

# En iyi pratikler

- ✅ **DTO doğrula, entity’yi değil.** (API sözleşmesi DTO’dur.)
- ✅ **İş kurallarını** validator’a değil **Service** katmanına koy. (Validator = girdi doğrulama)
- ✅ **Hataları alan bazında** ver; kullanıcı hangi alanı düzelteceğini anlasın.
- ✅ **Asenkron kontrollerde** `MustAsync` kullan; DB erişimleri bloklamasın.
- ✅ **Tekrar eden kuralları** `Include` veya base validator ile paylaş.
- ✅ **Localization** istersen `WithName()`/`WithMessage()` + kaynak dosyaları kullan.
- ✅ **Koleksiyon/nested DTO** için `RuleForEach`/`SetValidator` kullan.
- ✅ Swagger’da **örnek istek/cevap** ve hata şemasını tanımla (kullanıcı deneyimi).

# Sık sorulanlar

- **Data Annotations ile farkı?**
    
    Attribute’lar basit işler için iyi; FluentValidation kompleks kurallarda çok daha güçlü, test edilebilir ve temiz.
    
- **Controller’da `ModelState.IsValid` yazmalı mıyım?**
    
    `AddFluentValidationAutoValidation()` + `[ApiController]` varsa **gerekmez**; otomatik 400 döner.
    
- **Create/Update aynı kurallar?**
    
    Ortak kuralları `Include(...)` ile paylaş, farklı olanları ilgili validator’a ekle.
    

---

## Mini örnek (uçtan uca)

**Controller**

```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateHotelDto dto)
{
    // Hatalıysa otomatik 400 döndü; buraya gelmişse geçerli demektir.
    var hotel = await _hotelService.AddAsync(dto);
    return CreatedAtAction(nameof(GetById), new { id = hotel.Id }, hotel);
}

```

**Service (özet)**

```csharp
public async Task<HotelDto> AddAsync(CreateHotelDto dto)
{
    // burada artık dto güvenli ve kurallara uygun
    var entity = _mapper.Map<Hotel>(dto);
    await _repository.AddAsync(entity);
    return _mapper.Map<HotelDto>(entity);
}

```

---

kısacası:

- **FluentValidation** = **DTO doğrulama motorun**
- **Artıları**: okunaklı kurallar, güçlü senaryolar, test kolaylığı, otomatik 400
- **Yerleşimi**: DTO yanında `Validator` sınıfları, Program.cs’de otomatik devreye alma

