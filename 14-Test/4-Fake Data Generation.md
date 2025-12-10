
Şimdi Unit Test yazarken yaşanan **"Arrange (Hazırlık) Yorgunluğu"**nu bitiren kütüphaneye, **AutoFixture**'a geliyoruz.

Çoğu geliştirici AutoFixture'ı "Rastgele veri üreten bir araç" (Bogus gibi) zanneder. Oysa AutoFixture'ın felsefesi **"Anonymous Variables" (Anonim Değişkenler)** kavramıdır.

Bir Mimar olarak AutoFixture'ı; **Boilerplate Code'u yok etme**, **Döngüsel Referans (Circular Reference) Krizleri** ve **xUnit Entegrasyonu (`[AutoData]`)** üzerinden derinlemesine inceleyelim.

---

### 1. Sorun: "Arrange" Cehennemi

Standart bir test yazarken, testin asıl mantığıyla (Act) ilgisi olmayan onlarca satır kod yazarız.

**Eski Usül (Amelelik):**

C#

```cs
[Fact]
public void User_Should_Register()
{
    // Arrange: Sadece bu kısım 10 satır!
    var user = new User();
    user.Id = 1;
    user.Name = "Test Name";
    user.Email = "test@test.com";
    user.Address = new Address { City = "Istanbul", Zip = "34000" };
    user.Orders = new List<Order>();
    // ... daha 50 tane property olsa tek tek doldurman gerekir.

    // Act
    _service.Register(user);
}
```

Eğer `User` sınıfına yeni bir zorunlu alan (Required) eklenirse, projedeki **1000 tane test patlar** ve hepsini tek tek düzeltmen gerekir.

---

### 2. Çözüm: AutoFixture (Anonim Veri)

AutoFixture der ki: _"Testin bu verinin 'Ahmet' veya 'Mehmet' olmasıyla ilgilenmiyorsa, bana bırak ben doldurayım."_

**AutoFixture Usulü:**

C#

```cs
[Fact]
public void User_Should_Register()
{
    var fixture = new Fixture();
    
    // Tek satırda, tüm propertyleri (iç içe nesneler dahil) doldurur!
    var user = fixture.Create<User>(); 

    _service.Register(user);
}
```

**Sonuç:**

- `user.Name` -> "Name82734-guid..." (Rastgele string)
    
- `user.Age` -> 42 (Rastgele int)
    
- `user.Address` -> Dolu bir Address nesnesi.
    

Artık `User` sınıfına yeni alan eklense bile testin kırılmaz. AutoFixture onu da otomatik doldurur.

---

### 3. Mühendislik Harikası: `[AutoData]` (xUnit Entegrasyonu)

Bu özellik, kod kalitesini zirveye taşır.

Test metodunun içinde var fixture = new Fixture() yazmak bile tekrardır.

`AutoFixture.Xunit2` paketi ile test metodu parametrelerini otomatik doldurabilirsin.

C#

```cs
[Theory, AutoData] // Sihirli Attribute
public void Register_ShouldReturnTrue(User user, int orderCount)
{
    // 'user' ve 'orderCount' parametreleri AutoFixture tarafından doldurulup gönderildi!
    // Arrange fazı tamamen yok oldu.
    
    var result = _service.Register(user);
    Assert.True(result);
}
```

---

### 4. Özelleştirme: `Build`, `With`, `Without`

Bazen "Anonim" veri yetmez. _"User dolu gelsin AMA yaşı 18 olsun"_ demek istersin.

C#

```cs
var user = fixture.Build<User>()
    .With(x => x.Age, 18)        // Yaşı sabitle
    .Without(x => x.Orders)      // Siparişleri boş bırak (Null değil, atlama yap)
    .Do(x => x.IsActive = true)  // İşlem yap
    .Create();
```

---

### 5. Kritik Mühendislik Sorunu: Circular Reference (StackOverflow)

AutoFixture kullanırken Junior'ların %90'ının tosladığı duvar şudur: **Entity Framework Nesneleri.**

- **Senaryo:** `User` sınıfının içinde `List<Order>` var. `Order` sınıfının içinde de `User` var (Navigation Property).
    
- **Olay:**
    
    1. AutoFixture `User` yaratır.
        
    2. İçindeki `Orders` listesini doldurmaya çalışır.
        
    3. `Order` yaratırken içindeki `User`'ı doldurmaya çalışır.
        
    4. Tekrar `User`... Tekrar `Order`...
        
    5. **Sonuç:** `StackOverflowException`. Sonsuz döngü.
        

Çözüm (OmitOnRecursionBehavior):

Fixture'a "Kendini tekrar edersen dur" demen gerekir.

C#

```cs
fixture.Behaviors.Add(new OmitOnRecursionBehavior());
// Artık döngüsel referans algıladığında o property'i boş bırakır, patlamaz.
```

---

### 6. AutoFixture vs Bogus

Mülakatlarda sık sorulur: _"Bogus varken neden AutoFixture?"_

- **Bogus:** **"Fake Data"** üretir. Gerçekçi veriler lazım olduğunda kullanılır. (UI Testleri, Demo verisi).
    
    - `faker.Internet.Email()` -> "george@gmail.com" üretir.
        
- **AutoFixture:** **"Anonymous Data"** üretir. Yapısal testler (Unit Test) içindir.
    
    - `fixture.Create<string>()` -> "Email-GUID-23423" üretir.
        

**Mühendislik Kararı:** Unit Testlerde **AutoFixture** kullanılır çünkü verinin içeriği önemsizdir. E2E testlerde veya veritabanı seed ederken **Bogus** kullanılır.

---
