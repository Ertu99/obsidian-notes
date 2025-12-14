Microsoft's Built-in DI Container'ın (MEDI) "hafif ve standart" olduğunu, Autofac'in ise "gelişmiş özellikler" sunduğunu konuşmuştuk.

Peki, **Autofac kadar ağır bir kütüphane kurmadan**, Microsoft'un kendi Container'ına o eksik olan süper güçleri (Assembly Scanning, Decorator Pattern) eklemenin bir yolu yok mu?

İşte **Scrutor** tam olarak budur. O bir DI Container değildir; Microsoft DI Container'ına takılan bir **Turbo Şarj** kitidir. Andrew Lock (ünlü .NET blog yazarı) gibi isimlerin şiddetle önerdiği, projedeki "ameleliği" bitiren kütüphanedir.

Bu konuyu **"Convention over Configuration" (Konfigürasyon yerine Kural)** prensibi ve **Open/Closed Principle** üzerinden derinlemesine inceleyelim.

---

### 1. Sorun: "Manual Registration Hell" (Kayıt Cehennemi)

Büyük bir projede (Monolit veya Mikroservis) 100 tane servis ve repository olabilir.

Program.cs dosyasını açıp tek tek şunları yazmak bir mühendislik eziyetidir:

C#

```cs
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IProductRepository, ProductRepository>();
services.AddScoped<IOrderRepository, OrderRepository>();
// ... 97 satır daha ...
```

**Risk:** Yeni bir `PaymentRepository` yazdın ama buraya eklemeyi unuttun. Kod derlenir (Compile Time), ama çalıştırınca (Runtime) "Service Not Found" hatası alırsın.

### 2. Çözüm: Assembly Scanning (Otomatik Tarama)

Scrutor'un en büyük özelliği, metinde de geçtiği gibi _"automatically scan assemblies for services"_.

Sen Scrutor'a bir **Kural (Convention)** verirsin, o gidip DLL'leri tarar ve kurala uyan her şeyi otomatik kaydeder.

**Mühendislik Kodu:**

C#

```cs
builder.Services.Scan(scan => scan
    .FromCallingAssembly() // 1. Nereye bakayım? (Bu projenin DLL'ine)
    .AddClasses(classes => classes.Where(type => type.Name.EndsWith("Repository"))) // 2. Kimi seçeyim? (Sonu 'Repository' bitenleri)
    .AsImplementedInterfaces() // 3. Nasıl kaydedeyim? (Interface'i ile eşleştir)
    .WithScopedLifetime()); // 4. Ömrü ne olsun? (Scoped)
```

Bu tek blok kod, şu anki ve gelecekteki **tüm** repository'leri otomatik kaydeder. Yeni bir sınıf eklediğinde `Program.cs`'e dokunmana gerek kalmaz.

---

### 3. Decorator Pattern (Mühendislik Zirvesi)

Scrutor'un az bilinen ama en kritik yeteneği, Microsoft DI'da yapılması çok zor olan **Decorator** desteğidir.

**Senaryo:** `UserService` sınıfın var. Bunun metodları çalışmadan önce ve sonra loglama yapmak veya sonucu Cache'lemek istiyorsun. Ama `UserService` kodunu değiştirmek istemiyorsun (**Open/Closed Principle**: Gelişime açık, değişime kapalı).

**Scrutor ile Çözüm:**

1. `UserService` (Asıl işi yapan).
    
2. `CachedUserService` (Decorator - Asıl servisi sarmalar).
    

C#

```cs
// Scrutor ile tek satırda dekorasyon
builder.Services.Decorate<IUserService, CachedUserService>();
```

Bunu yaptığında;

- Uygulamanın herhangi bir yerinde `IUserService` istendiğinde, Container otomatik olarak **`CachedUserService`** verir.
    
- `CachedUserService` içinde de asıl `UserService` çalışır.
    
- Koduna hiç dokunmadan araya Cache katmanı eklemiş olursun.
    

---

### 4. Fluent API ve Esneklik

Metinde geçen _"fluent API that makes it easy to configure services"_ kısmı, kodun okunabilirliğini artırır.

Örneğin, sadece public olan sınıfları değil, internal sınıfları da taratabilirsin (ki bu genelde manuel kayıtta unutulur).

Veya bir servisi hem IUserService hem de IBaseService olarak aynı anda kaydedebilirsin (AsSelfWithInterfaces).

---

### 5. Mühendislik Riski: Startup Performance (Reflection Maliyeti)

Scrutor, uygulama ilk ayağa kalkarken (Startup) **Reflection** kullanır. Yani DLL dosyalarını diskten okur, içindeki tüm tipleri tek tek analiz eder.

- **Küçük/Orta Proje:** Fark edilmez (milisaniyeler sürer).
    
- **Devasa Proje (Binlerce Class):** Startup süresini 1-2 saniye uzatabilir.
    
- **Serverless (AWS Lambda):** "Cold Start" süresi kritik olduğu için Serverless ortamlarında Scrutor (veya genel olarak Assembly Scanning) kullanımı dikkatle ölçülmelidir.
    

---

