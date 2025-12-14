
Attığın metin aslında DI Container'ın anatomisini çok güzel özetlemiş: **"Configuration API"** ve **"Resolution API"** olmak üzere iki ana parçadan oluşur. Bir Mühendis olarak bu iki parçanın .NET Core'daki karşılıklarını ve arka plandaki **"Object Graph" (Nesne Grafiği)** oluşturma yeteneğini anlamalısın.

Gel bu "Kutuyu" (Container) açalım ve içindeki çarklara bakalım.

---

### 1. DI Container Nedir? (Analoji: Otomobil Fabrikası)

Eskiden (DI yokken) her sınıf kendi bağımlılığını yaratırdı (`new Lastik()`). Bu, her müşterinin arabasını almak için önce lastik fabrikasına gidip lastik alması, sonra motor fabrikasından motor alıp birleştirmesi gibiydi.

DI Container, tam teşekküllü bir Otomobil Fabrikasıdır.

Sen sadece "Bana bir Araba ver" dersin. Fabrika şunları otomatik yapar:

1. Araba için Lastik lazım. Lastik reyonuna git.
    
2. Lastik için Kauçuk lazım. Depoya git.
    
3. Kauçuğu al -> Lastiği üret -> Arabayı üret.
    
4. Anahtarı sana teslim et.
    

Bu zincirleme üretim sürecine **Auto-Wiring (Otomatik Bağlama)** denir.

---

### 2. Parça 1: Configuration API (Kayıt Masası)

Metinde _"used to register the types"_ denen kısım. .NET Core'da bunun karşılığı **`IServiceCollection`** arayüzüdür.

Uygulama henüz ayağa kalkarken (`Program.cs`), fabrikanın üretim planını burada hazırlarsın. Container'a **"Hangi tür istendiğinde, hangi sınıfı oluşturayım?"** bilgisini verirsin.

C#

```csharp
// Burası Configuration API'dır
var builder = WebApplication.CreateBuilder(args);

// "Biri senden IAraba isterse, ona Togg ver."
builder.Services.AddScoped<IAraba, Togg>(); 

// "Biri senden IMotor isterse, ona ElektrikliMotor ver."
builder.Services.AddSingleton<IMotor, ElektrikliMotor>();
```

**Mühendislik Notu:** Burası sadece tanımdır. Henüz hiçbir nesne (`new`) oluşturulmaz. Sadece tarif defterine yazılır.

---

### 3. Parça 2: Resolution API (Çözümleme/Teslimat)

Metinde _"used to retrieve instances"_ denen kısım. .NET Core'da bunun karşılığı **`IServiceProvider`** arayüzüdür.

Uygulama çalışmaya başladığında, nesnelerin talep edildiği ve üretildiği andır.

**İki şekilde çalışır:**

1. **Otomatik (Constructor Injection):** Controller'ın constructor'ında `IAraba` istersin. Container (ServiceProvider) devreye girer, `IAraba`'yı bulur, onun bağımlılıklarını (`IMotor`) bulur, hepsini üretip sana verir.
    
2. **Manuel (Service Locator - Tehlikeli):** Kendi elinle Container'dan nesne istersin.
    
    C#
    
    ```csharp
    // Resolution API'yi manuel kullanmak
    var araba = app.Services.GetRequiredService<IAraba>();
    ```
    

---

### 4. Object Graph (Nesne Grafiği) ve Recursive Resolution

DI Container'ın asıl gücü **Özyinelemeli (Recursive)** çözümleme yeteneğidir.

Senaryo: A -> B'ye, B -> C'ye, C -> D'ye bağımlı.

Sen Container'dan sadece A'yı istersin.

Container şöyle çalışır:

1. A'yı yaratmak için B lazım. B var mı? Yok. Bekle.
    
2. B'yi yaratmak için C lazım. C var mı? Yok. Bekle.
    
3. C'yi yaratmak için D lazım. D var mı? Var (Bağımlılığı yok).
    
4. **D'yi üret -> C'ye ver (C üretildi) -> B'ye ver (B üretildi) -> A'ya ver.**
    
5. A'yı teslim et.
    

Bu ağaca **Dependency Graph** denir. Container bunu milisaniyeler içinde çözer.

---

### 5. Yerleşik (Built-in) vs 3. Parti Containerlar

ASP.NET Core'un içinde gömülü gelen (Built-in) DI Container oldukça yeteneklidir ve projelerin %95'ine yeter.

Ancak bazı ileri seviye özellikler eksiktir. Bu yüzden **Autofac**, **Ninject**, **Castle Windsor** gibi 3. parti kütüphaneler kullanılır.

**Neler Eksik? (Autofac ne katar?)**

- **Property Injection:** .NET Container sadece Constructor Injection destekler. Property injection için 3. parti gerekir.
    
- **Assembly Scanning:** "Şu klasördeki isminde 'Service' geçen tüm sınıfları otomatik kaydet" diyebilmek (`Scrutor` kütüphanesi ile de yapılır).
    
- **Interceptor (AOP):** Metotların arasına çalışma zamanında girmek (Filtre mantığı) için dinamik proxy oluşturma.
    

---
### 6. Mühendislik Tuzağı: Service Locator Anti-Pattern

Bu, DI Container kullanırken yapılan **en büyük mimari hatadır.**

**Senaryo:** Bir sınıfın constructor'ına 10 tane servis yazmak zor geldi. Bunun yerine constructor'a sadece Container'ın kendisini (`IServiceProvider`) aldın. Ve metotların içinde ihtiyacın oldukça `_provider.GetService<IUserManager>()` diye çağırdın.

C#

```csharp
public class KotuSinif
{
    private readonly IServiceProvider _provider; // Koca fabrikayı içeri aldın!

    public KotuSinif(IServiceProvider provider)
    {
        _provider = provider;
    }

    public void IslemYap()
    {
        // GİZLİ BAĞIMLILIK (Hidden Dependency)
        var db = _provider.GetService<DbContext>(); 
        // ...
    }
}
```

**Neden Kötü?**

1. **Bağımlılıklar Gizlenir:** Bu sınıfın çalışması için `DbContext` gerektiği constructor'a bakınca anlaşılmaz. Çalıştırınca "Null Reference" hatası alırsın.
    
2. **Test Edilemez:** Test yazarken içeriye tüm Container'ı mock'laman gerekir, bu çok zordur.
    
3. **Kural:** Container, sadece en tepede (Composition Root - yani Controller veya Program.cs) kullanılmalıdır. Sınıfların içine sızdırılmamalıdır.
