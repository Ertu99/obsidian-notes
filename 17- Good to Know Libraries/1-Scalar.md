
İlk konuğumuz, .NET dünyasında yıllardır süren **Swagger UI** hegemonyasını yıkan, API dokümantasyonunun yeni ve modern yüzü: **Scalar**.

Yıllardır API geliştirirken Swashbuckle kurup o klasik, mavi/yeşil ve hantal Swagger arayüzüne baktık.

Ancak .NET 9 ile birlikte Microsoft, Swashbuckle'ı varsayılan şablonlardan kaldırdı. Meydan boş kaldı.

İşte Scalar tam bu noktada, API dokümantasyonunu sadece "okunacak bir metin" olmaktan çıkarıp, **"Etkileşimli Bir Deneyim"e (Developer Experience - DX)** dönüştüren araçtır.

Bu konuyu; **OpenAPI vs UI Ayrımı**, **Entegrasyon**, **Swagger/Redoc Karşılaştırması** ve **Client Code Generation** yetenekleri üzerinden eksiksiz inceleyelim.

---

### 1. Felsefe: Generator vs Renderer (Kavram Kargaşası)

Önce şu ayrımı netleştirelim. Çoğu kişi Swagger UI'ı API dökümanını "yaratan" şey sanır. Yanlıştır.

1. **OpenAPI Specification (Eski adıyla Swagger Spec):** API'nin haritasıdır (JSON dosyası). Hangi endpoint var, parametresi ne, dönüş tipi ne?
    
    - _Sorumlusu:_ .NET'in kendisi (`Microsoft.AspNetCore.OpenApi`) veya `Swashbuckle.AspNetCore`.
        
2. **Documentation UI (Arayüz):** O JSON dosyasını okuyup, insanların anlayacağı, butonlara basabileceği web sitesine çeviren araçtır.
    
    - _Eski Kral:_ Swagger UI.
        
    - _Yeni Kral:_ **Scalar.**
        

Scalar, JSON üretmez. **Var olan JSON'ı alır ve onu "Ferrari" gibi gösterir.**

---

### 2. Neden Scalar? (Swagger UI'dan Farkı Ne?)

Bir Mimar olarak takımına neden "Swagger UI'ı çöpe atıp Scalar'a geçelim" dersin?

|**Özellik**|**Swagger UI**|**Redoc**|**Scalar**|
|---|---|---|---|
|**Görünüm**|2010'lardan kalma, demode|Modern, temiz|**Ultra Modern, Dark Mode, Özelleştirilebilir**|
|**Hız**|Büyük API'lerde (500+ endpoint) tarayıcıyı kilitler (Freeze)|Hızlıdır (Static)|**Çok Hızlı** (Vue.js tabanlı)|
|**Etkileşim**|"Try it out" butonu var|Yok (Sadece okuma - Read Only)|**Var ve çok gelişmiş (Postman gibi)**|
|**Kod Üretimi**|Yok|Yok|**VAR (JavaScript, Python, C#, Go...)**|
|**Arama**|Yavaş (Ctrl+F hissi verir)|İyi|**Command Palette (Cmd+K) ile anında arama**|

---

### 3. Entegrasyon: .NET Core ile Kurulum

Scalar, .NET ekosistemiyle derinlemesine entegre olan **`Scalar.AspNetCore`** paketini sunar.

**Kurulum:**

1. Paketi yükle: `dotnet add package Scalar.AspNetCore`
    
2. `Program.cs` ayarını yap:
    

C#

```cs
var builder = WebApplication.CreateBuilder(args);

// 1. OpenAPI Generator'ı ekle (.NET'in kendi motoru)
builder.Services.AddOpenApi(); 

var app = builder.Build();

// 2. OpenAPI JSON endpoint'ini aç (/openapi/v1.json)
app.MapOpenApi();

// 3. Scalar Arayüzünü aç (/scalar/v1)
app.MapScalarApiReference(options =>
{
    options
        .WithTitle("My E-Commerce API")
        .WithTheme(ScalarTheme.Mars) // Tema Seçimi
        .WithSidebar(true)
        .WithDownloadButton(true)
        .WithDarkModeToggle(true);
});

app.Run();
```

Artık projeyi çalıştırıp `/scalar/v1` adresine gittiğinde, o modern arayüzü görürsün.

---

### 4. Mühendislik Gücü: Client Code Generation

Scalar'ın en büyük satış noktası (Selling Point) budur.

API'ni kullanacak olan Frontend geliştiricisi veya Mobil geliştiricisi Scalar arayüzünü açar.

Bir endpoint'e tıklar (örn: POST /api/orders).

Sağ tarafta "Client Libraries" sekmesini görür.

- "Ben Axios kullanıyorum" der -> Scalar ona **hazır Axios kodunu** verir.
    
- "Ben C# HttpClient kullanıyorum" der -> Hazır C# kodunu verir.
    
- "Ben Python Requests kullanıyorum" der -> Hazır Python kodunu verir.
    

**Fayda:** İletişim maliyetini sıfıra indirir. "Abi bunu nasıl çağırıyorduk?" sorusu biter.

---

### 5. Test Client (Tarayıcı İçinde Postman)

Swagger UI'daki "Try it out" butonu bazen yetersiz kalır.

Scalar, arayüzün içine gömülü tam teşekküllü bir HTTP Client sunar.

- Header'ları değiştirebilirsin.
    
- Body'yi JSON olarak düzenleyebilirsin.
    
- Authentication (Bearer Token) ekleyebilirsin.
    
- Yanıtı (Response) renklendirilmiş JSON olarak görürsün.
    
- Yanıt süresini ve boyutunu analiz edersin.
    

Yani basit testler için Postman açmana gerek kalmaz.

---

### 6. Hosting Stratejisi: CDN vs Embedded

Scalar'ı projenin içine gömmek (`Scalar.AspNetCore` ile) en kolayıdır. Ancak büyük kurumsal yapılarda bazen dokümantasyonun koddan ayrı yaşaması istenir.

1. **Embedded (Gömülü):** Proje çalışırken arayüz de sunulur. Development ortamı için harikadır.
    
2. **CDN / Static Hosting:**
    
    - CI/CD pipeline'ında `openapi.json` dosyasını üretirsin.
        
    - Bu JSON'ı Scalar'ın JavaScript CDN kütüphanesiyle birleştirip statik bir HTML dosyası yaparsın.
        
    - Bunu GitHub Pages veya Azure Static Web Apps'e atarsın.
        
    - **Fayda:** API sunucun kapalı olsa bile dokümantasyonun her zaman yayında kalır. Scalar buna tam destek verir.
        

---
