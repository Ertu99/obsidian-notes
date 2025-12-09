 Microsoft'un kendi DI Container'ını (MEDI) öğrendik. O, "standart donanım" bir araba gibidir; işini görür, sizi A noktasından B noktasına götürür.

Ama **Autofac**, "modifiyeli bir yarış arabasıdır".

Microsoft'un kütüphanesi "Conforming Container" (Standartlara Uyan) prensibiyle, basit ve hafif olması için tasarlanmıştır. Ancak proje büyüdükçe (500+ servis), gelişmiş özelliklere (AOP, Property Injection, Module Based Registration) ihtiyaç duyarsın. İşte o noktada Autofac devreye girer.

Autofac'i sadece "başka bir kütüphane" olarak değil, **"Yazılım Mimarisi Yeteneklerini Artıran Bir Araç"** olarak inceleyeceğiz.

---

### 1. Autofac Nedir ve Neden İhtiyaç Duyarız?

Autofac, .NET için geliştirilmiş açık kaynaklı bir DI framework'üdür. Nesnelerin ve bağımlılıkların yaşam döngüsünü otomatik olarak yöneterek uygulamayı daha yönetilebilir kılar.

Microsoft DI ile yapamadığın (veya çok zor yaptığın) şu mühendislik çözümlerini Autofac ile "tek satırda" yaparsın:

1. **Assembly Scanning (Otomatik Tarama):** Microsoft DI'da her servisi tek tek `AddScoped` ile eklemen gerekir. Autofac'te "Şu DLL içindeki sonu 'Service' ile biten her şeyi kaydet" diyebilirsin.
    
2. **Property Injection:** Microsoft sadece Constructor Injection destekler. Autofac, property'lere de değer atayabilir (Legacy kodlar için hayat kurtarıcıdır).
    
3. **AOP (Aspect Oriented Programming) & Interceptors:** Metotların arasına (çalışmadan önce/sonra) kod sokuşturmak (Loglama, Transaction yönetimi) Autofac ile çok kolaydır.
    
4. **Modules (Modüler Yapı):** `Program.cs` dosyasının 1000 satır olmasını engeller. Her modül kendi kayıtlarını (Registration) yönetir.
    

---

### 2. Mekanizma: `ContainerBuilder`

Microsoft DI'da `IServiceCollection` vardı, Autofac'te ise **`ContainerBuilder`** vardır.

Süreç şöyledir:

1. **Builder Oluştur:** `var builder = new ContainerBuilder();`.
    
2. **Kayıt (Registration):** `builder.RegisterType<MyService>().As<IService>();`.
    
3. **İnşa (Build):** `var container = builder.Build();`. Bu işlem sana `IContainer` arayüzünü döner, bu da Microsoft'taki `IServiceProvider`'ın karşılığıdır.
    

---

### 3. Kayıt Yöntemleri ve Yaşam Döngüleri (Dil Farkı)

Autofac'in jargonu Microsoft'tan biraz farklıdır. Bir Backend Mühendisi olarak bu sözlüğü bilmelisin:

|**Microsoft DI**|**Autofac Karşılığı**|**Anlamı**|
|---|---|---|
|`AddTransient`|`.InstancePerDependency()`|Her istekte yeni bir tane.|
|`AddScoped`|`.InstancePerLifetimeScope()`|HTTP isteği (Scope) boyunca aynı.|
|`AddSingleton`|`.SingleInstance()`|Uygulama boyunca tek.|

**Kod Örneği:**

C#

```csharp
// Microsoft Tarzı
services.AddScoped<IUserRepository, UserRepository>();

// Autofac Tarzı
builder.RegisterType<UserRepository>()
       .As<IUserRepository>()
       .InstancePerLifetimeScope();
```

---

### 4. Gelişmiş Özellik 1: Modüller (Modules)

Büyük projelerde `Program.cs` dosyası şişer. Autofac ile kayıt işlemlerini dosyalara bölebilirsin.

C#

```cs
// ServiceModule.cs
public class ServiceModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        // Sadece Servis katmanının kayıtları burada
        builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly())
               .Where(t => t.Name.EndsWith("Service"))
               .AsImplementedInterfaces()
               .InstancePerLifetimeScope();
    }
}

// Program.cs
builder.RegisterModule(new ServiceModule());
builder.RegisterModule(new RepositoryModule());
```

Bu sayede her katman (Layer) kendi bağımlılıklarını kendisi yönetir. Mühendislikte buna **"Separation of Concerns"** denir.

---

### 5. Gelişmiş Özellik 2: Interceptors (AOP - Mühendislik Zirvesi)

Burası Autofac'in şov yaptığı yerdir.

Senaryo: Projedeki 50 tane metodun hepsi çalışmadan önce "Loglama" yapmak istiyorsun.

- **Microsoft DI:** 50 metodun içine tek tek `Logger.Log()` yazarsın.
    
- **Autofac:** Bir `LogInterceptor` sınıfı yazarsın ve Autofac'e "Tüm servisleri bu interceptor ile sarmala" dersin.
    

Autofac, çalışma zamanında (Runtime) senin sınıflarından miras alan sanal sınıflar (Proxy) üretir ve senin koduna dokunmadan araya girer.

---

### 6. ASP.NET Core Entegrasyonu

Autofac'i .NET Core projesine entegre etmek için `Program.cs` içinde Factory'yi değiştirmen gerekir.

C#

```cs
// 1. Varsayılan Factory'yi Autofac ile değiştir
host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

// 2. Autofac konfigürasyonunu yap
host.ConfigureContainer<ContainerBuilder>(builder =>
{
    builder.RegisterModule(new MyModule());
});
```

Bunu yaptığında, Microsoft'un kendi Container'ı devre dışı kalır, direksiyona Autofac geçer.

---
