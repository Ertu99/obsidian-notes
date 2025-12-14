
Şimdi kodun "Bekçisi"ne, kirli verinin sistemin kalbine girmesini engelleyen **FluentValidation** kütüphanesine geliyoruz.

Çoğu Junior geliştirici, C# sınıflarının tepesine [Required] veya [StringLength] gibi Data Annotations (Attribute) ekleyerek validasyon yapar.

Bir Mimar için ise bu yöntem; Single Responsibility Principle (SRP) ihlalidir ve test edilebilirliği yok eder.

Bu konuyu; **Clean Architecture Uyumu**, **Complex & Async Validation**, **Cross-Property Rules** ve **Unit Testing** yetenekleri üzerinden, tek seferde ve eksiksiz inceleyelim.

---

### 1. Felsefe: Neden Attribute Kullanmıyoruz? (Separation of Concerns)

**Data Annotations (Eski Yöntem):**

C#

```cs
public class UserDto {
    [Required(ErrorMessage = "Ad lazım")]
    [MaxLength(50)]
    public string Name { get; set; }
}
```

- **Sorun 1:** DTO/Entity sınıfı kirlendi. İş kuralları (Validasyon) ile veri yapısı birbirine girdi.
    
- **Sorun 2:** Şartlı validasyon zordur. _"Eğer kullanıcı tipi 'Kurumsal' ise Vergi Numarası zorunlu olsun"_ kuralını Attribute ile yazamazsın.
    
- **Sorun 3:** Test edemezsin. Validasyon mantığını test etmek için tüm `UserDto`yu doldurman gerekir.
    

FluentValidation (Modern Yöntem):

Validasyon kuralları, ayrı bir sınıfta (UserValidator) tanımlanır. DTO tertemiz kalır.

---

### 2. Mimari: Validator Anatomisi

Her model için `AbstractValidator<T>` sınıfından türeyen bir sınıf yazarız.

C#

```cs
public class CreateUserValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserValidator()
    {
        // Temel Kurallar
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("E-posta boş olamaz")
            .EmailAddress().WithMessage("Geçerli bir e-posta girin");

        // Zincirleme (Chaining)
        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Matches("[A-Z]").WithMessage("En az 1 büyük harf içermeli");
    }
}
```

---

### 3. Mühendislik Harikası: Conditional & Cross-Property Validation

FluentValidation'ın en büyük gücü, "Duruma Göre Değişen" kuralları yönetebilmesidir.

**Senaryo:** "Eğer indirim kuponu varsa, kupon kodu boş olamaz."

C#

```cs
RuleFor(x => x.CouponCode)
    .NotEmpty()
    .When(x => x.HasCoupon); // Sadece bu şart sağlanırsa çalışır
```

**Senaryo:** "Bitiş tarihi, Başlangıç tarihinden büyük olmalı." (İki property arası ilişki).

C#

```cs
RuleFor(x => x.EndDate)
    .GreaterThan(x => x.StartDate)
    .WithMessage("Bitiş tarihi başlangıçtan önce olamaz.");
```

---

### 4. İleri Seviye: Custom Logic ve `Must`

Hazır kurallar (`NotEmpty`, `Equal`) yetmediğinde **`Must`** (Zorunda) metodu ile kendi C# mantığını yazabilirsin.

C#

```cs
RuleFor(x => x.Age)
    .Must(age => age >= 18)
    .WithMessage("18 yaşından küçükler giremez.");
```

Mühendislik Detayı (Arguments):

Must metodu tüm nesneye de erişebilir.

C#

```cs
RuleFor(x => x) // 'x' burada tüm nesnedir
    .Must(x => ValidateComplexLogic(x.Type, x.Category))
    .WithMessage("Kategori ve Tip uyumsuz.");
```

---

### 5. Kritik Konu: Async Validation ve Database Check

Mülakat sorusu: _"Kullanıcı kaydolurken e-postanın daha önce alınıp alınmadığını nasıl kontrol edersin?"_

Bunu Controller'da yapmak yanlıştır. Validasyon katmanında yapılmalıdır. Ancak veritabanına gitmek **Asenkron** bir işlemdir.

Çözüm: MustAsync

Validator sınıfının Constructor'ına Repository enjekte edersin (Dependency Injection).

C#

```cs
public class UserValidator : AbstractValidator<UserDto>
{
    private readonly IUserRepository _repo;

    public UserValidator(IUserRepository repo)
    {
        _repo = repo;

        RuleFor(x => x.Email)
            .MustAsync(async (email, cancellation) => 
            {
                bool exists = await _repo.EmailExistsAsync(email);
                return !exists; // Varsa false dön (Hata ver)
            })
            .WithMessage("Bu e-posta zaten kullanımda.");
    }
}
```

**Uyarı:** Bu kural her çalıştığında veritabanına sorgu atar. Performans için dikkatli kullanılmalıdır.

---

### 6. Collections (Koleksiyonlar)

Eğer `Order` içinde `List<OrderItem>` varsa, her bir item'ı doğrulamak için `foreach` döngüsü yazmana gerek yoktur.

C#

```cs
// Ana Validator
RuleForEach(x => x.Items).SetValidator(new OrderItemValidator());
```

FluentValidation otomatik olarak listeyi döner ve her eleman için alt validator'ı çalıştırır.

---

### 7. Unit Testing (TestHelper)

FluentValidation, kurallarını test etmen için harika bir **TestHelper** paketi sunar.

C#

```cs
[Fact]
public void Should_Have_Error_When_Email_Is_Empty()
{
    var validator = new UserValidator();
    var model = new UserDto { Email = "" }; // Hatalı model

    // Assert
    var result = validator.TestValidate(model);
    
    // Fluent Assertions benzeri sözdizimi
    result.ShouldHaveValidationErrorFor(x => x.Email);
}
```

Böylece "Acaba Regex doğru çalışıyor mu?" diye tüm sistemi ayağa kaldırmadan kural bazlı test yapabilirsin.

---

### 8. ASP.NET Core Entegrasyonu (Automatic Validation)

Bu kütüphaneyi projeye eklediğinde (`builder.Services.AddFluentValidationAutoValidation()`), ASP.NET Core'un varsayılan validasyon motorunun yerine geçer.

1. İstek Controller'a gelir.
    
2. Action çalışmadan **önce** FluentValidation devreye girer.
    
3. Hata varsa, Controller kodu **HİÇ ÇALIŞMAZ.**
    
4. Otomatik olarak `400 Bad Request` ve hataların listesi (JSON) döner.
    

Bu sayede Controller içinde `if (!ModelState.IsValid)` yazmaktan kurtulursun.

---
