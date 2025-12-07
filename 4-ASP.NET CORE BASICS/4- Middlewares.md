Harika. MVC ve API mimarilerini kurduk. Şimdi bu yapıların içine giren trafiği (Request) ve çıkan cevabı (Response) nasıl yönettiğimizi, yani **Middleware (Ara Katman Yazılımı)** konusunu konuşacağız.

ASP.NET Core'un en güçlü ve esnek olduğu yer burasıdır. Bu konuyu sadece "araya giren kod" olarak değil, **bir boru hattı mühendisliği (Pipeline Engineering)** olarak ele alacağız.

---

### 1. Middleware Nedir? (Analoji: Su Arıtma Tesisi)

Geleneksel web sunucularında (eski IIS gibi), bir istek geldiğinde tek bir devasa blok (Monolitik) çalışırdı. ASP.NET Core'da ise işler modülerdir.

Uygulamanı bir **Su Arıtma Tesisi** gibi düşün:

1. Şebekeden kirli su gelir (**Request**).
    
2. Önce kaba filtreye girer (Taşlar ayrılır).
    
3. Sonra klorlama ünitesine girer (Mikroplar ölür).
    
4. Sonra ince filtreye girer.
    
5. En son temiz su deposuna (**Controller/Action**) ulaşır.
    

İşte bu filtrelerin her biri bir **Middleware**'dir. Su (Request) sırayla hepsinden geçer. Eğer klorlama ünitesinde bir sorun çıkarsa (örneğin su çok kirliyse), sistem suyu orada keser ve geri gönderir. Diğer aşamalara geçilmez.

---

### 2. Pipeline (Boru Hattı) Mimarisi

Middleware'lerin en kritik özelliği **sıralı (sequential)** çalışmasıdır.

`Program.cs` dosyasında yazdığın `app.Use...` komutlarının sırası hayati önem taşır.

**İşleyiş Mantığı (Rus Matruşka Bebekleri):**

1. İstek gelir, **Middleware A** onu karşılar. İşini yapar ve `next()` diyerek topu **Middleware B**'ye atar.
    
2. **Middleware B** işini yapar, `next()` der, **Middleware C**'ye atar.
    
3. En sonda asıl iş yapılır (Controller).
    
4. Cevap oluşur. Bu sefer **Geriye Dönüş** başlar.
    
5. **Middleware C** cevabı görür, gerekirse değiştirir, B'ye verir.
    
6. **B** A'ya verir.
    
7. **A** kullanıcıya gönderir.
    

Yani Middleware, hem **girişte** hem **çıkışta** çalışır.

---

### 3. Kritik Middleware Sıralaması (Order Matters)

Bir mühendis olarak sıralamayı yanlış yaparsan güvenlik açığı yaratırsın.

**Doğru Sıralama:**

1. `UseExceptionHandler` (Hata yakalayıcı en dışta olmalı ki içeride ne patlarsa patlasın yakalasın).
    
2. `UseHttpsRedirection` (Güvenli hatta zorla).
    
3. `UseStaticFiles` (Resim/CSS dosyaları. Buna kimlik sormaya gerek yok, o yüzden Auth'dan önce).
    
4. `UseRouting` (Nereye gideceğini bul).
    
5. **`UseAuthentication` (Kimsin?)**
    
6. **`UseAuthorization` (Yetkin var mı?)**
    
7. `MapControllers` (Son durak).
    

Felaket Senaryosu (Yanlış Sıralama):

Eğer UseAuthorization'ı UseAuthentication'dan önce yazarsan ne olur?

Sistem önce "Yetkin var mı?" diye sorar. Ama henüz "Kimsin?" sorusunu sormamıştır (Authentication çalışmadı). Dolayısıyla sistem herkesi yetkisiz sanar veya hata verir.

---

### 4. Middleware Türleri (`Run`, `Use`, `Map`)

Middleware'leri eklemenin 3 yolu vardır:

1. **`app.Use()`:** Zincirin devam etmesini sağlar. İşini yapar ve bir sonraki middleware'i çağırır (`next.Invoke()`).
    
2. **`app.Run()`:** **Terminal Middleware (Son Durak).** Zinciri koparır. Bundan sonra yazılan middleware'ler asla çalışmaz. Genelde en sonda kullanılır.
    
3. **`app.Map()`:** Yola göre dallanma yapar. "Eğer URL `/admin` ile başlıyorsa şu middleware zincirine gir, yoksa diğerinden devam et" demeni sağlar.
    

---

### 5. Özel (Custom) Middleware Yazmak

Her zaman hazır middleware'ler yetmez. Kendi "Loglama" middleware'imizi yazalım.

C#

```csharp
// Program.cs içinde
app.Use(async (context, next) =>
{
    // 1. REQUEST ANI (İstek gelirken)
    Console.WriteLine($"İstek Geldi: {context.Request.Path}");
    var watch = System.Diagnostics.Stopwatch.StartNew();

    // Zinciri devam ettir (Sıradaki middleware'i çağır)
    await next(); 

    // 2. RESPONSE ANI (Cevap dönerken)
    watch.Stop();
    Console.WriteLine($"İşlem Bitti. Süre: {watch.ElapsedMilliseconds}ms");
});
```

Bu kod ne yapar? Her isteğin ne kadar sürede tamamlandığını ölçer. Controller çalışıp bittikten sonra tekrar buraya (`await next()` satırının altına) döner ve süreyi yazar.

---

### 6. Short-Circuiting (Kısa Devre / Zinciri Kırma)

Bazen isteği yarı yolda kesmen gerekir.

**Örnek:** Sadece Chrome tarayıcısı kullananlara izin veren bir middleware.

C#

```csharp
app.Use(async (context, next) =>
{
    var userAgent = context.Request.Headers["User-Agent"].ToString();
    
    if (!userAgent.Contains("Chrome"))
    {
        // KISA DEVRE! next() çağrılmadı.
        // İstek buradan geri döner, Controller'a asla ulaşamaz.
        await context.Response.WriteAsync("Sadece Chrome kullanabilirsiniz.");
        return; 
    }

    await next(); // Sorun yok, devam et.
});
```

Bu mekanizma, kötü niyetli botları engellemek veya IP banlamak için kullanılır. Kaynak tüketmeden (Controller çalışmadan) kapıdan çevirirsin.

---

Middleware'lerin **çift yönlü (Request/Response)** çalışma mantığını ve sırasını test edelim.

**Senaryo:** `Program.cs` dosyasında 3 tane özel middleware ekledin (Sırasıyla A, B ve C). Her biri ekrana bir harf yazıyor ve sonra `next()` diyor. En sonda ise Controller var.

Kod şuna benziyor:

C#

```csharp
// Middleware A
app.Use(async (context, next) => {
    Console.Write("A1 ");
    await next();
    Console.Write("A2 ");
});

// Middleware B
app.Use(async (context, next) => {
    Console.Write("B1 ");
    await next();
    Console.Write("B2 ");
});

// Middleware C (Terminal - Run kullanmış)
app.Run(async context => {
    Console.Write("C ");
});
```

Bir istek geldiğinde konsolda hangi çıktı oluşur?



Buradaki `await next()` komutu aslında şu anlama gelir: **"Burada dur! Senden sonraki herkes işini bitirip geri dönene kadar bekle."**

Olay an be an şöyle gerçekleşir:

1. **GİRİŞ (Request Anı):**
    
    - **Middleware A** başlar -> Ekrana **`A1`** yazar.
        
    - `next()` çağrılır -> A burada donar bekler, top B'ye geçer.
        
    - **Middleware B** başlar -> Ekrana **`B1`** yazar.
        
    - `next()` çağrılır -> B burada donar bekler, top C'ye geçer.
        
2. **SON DURAK (Terminal):**
    
    - **Middleware C** çalışır -> Ekrana **`C`** yazar.
        
    - C biter ve geri döner (`return`).
        
3. **ÇIKIŞ (Response Anı - Geriye Dönüş):**
    
    - Top **Middleware B**'ye, tam donduğu yere (`next`in altına) geri döner.
        
    - B devam eder -> Ekrana **`B2`** yazar. B biter.
        
    - Top **Middleware A**'ya, tam donduğu yere geri döner.
        
    - A devam eder -> Ekrana **`A2`** yazar. A biter.
        

Sonuç: **A1 -> B1 -> C -> B2 -> A2**

Bu "U Dönüşü" yapısını anlamak, ileride "Exception Handling" (Hata Yakalama) middleware'lerinin neden en dışta olduğunu anlamanı sağlayacak (çünkü hatayı dönüş yolunda yakalarlar).