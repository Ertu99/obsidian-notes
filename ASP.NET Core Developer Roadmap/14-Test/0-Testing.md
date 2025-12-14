
Çoğu geliştirici için test yazmak "angarya" veya "zaman kaybı"dır.

Bir Mimar için ise Test; "Kodun Sigortasıdır". Değişiklik yaparken "Acaba bir yeri bozdum mu?" korkusunu yok eden tek şey, sağlam bir Test Suite'tir.

Attığın metin test türlerini (Unit, Integration, E2E) ve araçları (XUnit, Playwright vb.) listelemiş. Biz bu konuyu; **Test Piramidi**, **TestContainers** ile modern entegrasyon testleri ve **Mocking** stratejileri üzerinden, tek seferde ve tüm mühendislik derinliğiyle inceleyelim.

---

### 1. Büyük Resim: Test Piramidi (Strateji)

Test yazmak pahalı bir iştir. Neye ne kadar yatırım yapacağını **Mike Cohn'un Test Piramidi** belirler.

1. **Taban (Unit Tests - %70):** En hızlı, en ucuz, en çok sayıda. Tek bir fonksiyonu (Method) dış dünyadan izole edip test edersin.
    
2. **Orta (Integration Tests - %20):** Parçalar birbirine uyuşuyor mu? API, Veritabanına yazabiliyor mu?
    
3. **Tepe (E2E Tests - %10):** En yavaş, en pahalı. Tarayıcıyı aç, butona bas, sonucu gör. (Kullanıcı taklidi).
    

---

### 2. Unit Testing (Birim Testleri) ve İzolasyon

Amaç: "Benim yazdığım mantık (Business Logic) doğru mu?"

Burada veritabanı, dosya sistemi veya ağ yoktur. Hepsi sahtedir (Mock).

- **Araçlar:** .NET dünyasında standart **xUnit**'tir. (NUnit ve MSTest eskide kaldı).
    
- **Mocking:** Bağımlılıkları taklit etmek için **Moq** veya **NSubstitute** kullanılır.
    
- **AAA Deseni:** Arrange (Hazırla), Act (Çalıştır), Assert (Doğrula).
    

**Mühendislik Örneği (xUnit + Moq):**

C#

```cs
[Fact]
public void CalculateDiscount_ShouldReturn10Percent_WhenUserIsPremium()
{
    // Arrange (Hazırlık - Sahte bağımlılıklar)
    var mockUserRepo = new Mock<IUserRepository>();
    mockUserRepo.Setup(x => x.GetUser(1)).Returns(new User { IsPremium = true });
    
    var service = new PricingService(mockUserRepo.Object);

    // Act (Eylem)
    var result = service.CalculatePrice(100, userId: 1);

    // Assert (Doğrulama)
    Assert.Equal(90, result); // 100 TL -> 90 TL olmalı
}
```

---

### 3. Integration Testing (Entegrasyon) ve `WebApplicationFactory`

Amaç: "Kodum veritabanıyla/API ile doğru konuşuyor mu?"

Mock kullanmak yasaktır (veya minimumdur). Gerçek (veya gerçeğe çok yakın) ortam kullanılır.

ASP.NET Core'da bu iş için **`WebApplicationFactory`** mucizesi vardır.

- Bu sınıf, test çalışırken arka planda **In-Memory** bir test sunucusu (TestServer) ayağa kaldırır.
    
- Sana gerçek bir `HttpClient` verir.
    
- Sen bu client ile kendi API'ne istek atarsın (`client.GetAsync("/api/products")`).
    

Modern Yaklaşım: TestContainers

Eskiden test için "In-Memory Database" kullanılırdı ama bu gerçek SQL Server gibi davranmaz (Transaction hatalarını yakalamaz).

Şimdi TestContainers kütüphanesi kullanılıyor.

- Test başladığında Docker üzerinde **gerçek** bir SQL Server konteyneri ayağa kaldırır.
    
- Test bitince konteyneri çöpe atar.
    
- **Sonuç:** %100 güvenilir test ortamı.
    

---

### 4. E2E (Uçtan Uca) Testing ve Playwright

Amaç: "Kullanıcı siteyi açıp işlem yapabiliyor mu?"

Kodun içiyle (C#) ilgilenmez. Ekranda ne göründüğüyle ilgilenir.

- **Eski Kral:** Selenium (Yavaş, kırılgan, sürücü derdi var).
    
- **Yeni Kral:** **Playwright** (Microsoft yapımı). Çok hızlı, modern web'i (SPA/React) anlar, "Network Interception" yapabilir.
    

Senaryo: "Login butonuna basınca Ana Sayfa açılıyor mu?"

Playwright arka planda (Headless) veya görünür şekilde Chromium tarayıcısını açar, tıklar ve URL'i kontrol eder.

---

### 5. BDD (Behavior Driven Development)

Testleri yazılımcı diliyle (`Assert.Equal`) değil, İş Birimi (Business) diliyle (`Given-When-Then`) yazma sanatıdır.

- **Gherkin Dili:**
    
    Gherkin
    
    ```
    Scenario: Premium indirim hesaplama
      Given Kullanıcı Premium üyedir
      When 100 TL'lik ürün alırsa
      Then Fiyat 90 TL olmalıdır
    ```
    
- **Araçlar:** **SpecFlow** (En popüleri) veya **Cucumber**. Bu metni okur ve arkadaki C# kodunu çalıştırır.
    

---

### 6. Mühendislik Kuralı: Testable Code (Test Edilebilir Kod)

Test yazamıyorsan, sorun test aracında değil **Mimarindedir.**

- Eğer `new SqlConnection()` diyerek kodun içinde nesne yaratıyorsan (Tight Coupling), o kodu test edemezsin. Çünkü veritabanını "Mock"layamazsın.
    
- **Dependency Injection (DI)**, test edilebilirliğin anahtarıdır. Bağımlılıkları dışarıdan (Constructor) alırsan, test ederken sahtelerini verebilirsin.
    

---

### 7. Code Coverage (Kapsama Oranı) Tuzağı

Yöneticiler genelde "%80 Code Coverage olsun" diye baskı yapar.

- **Coverage:** Kodun kaç satırının test sırasında çalıştığını gösterir.
    
- **Tuzak:** %100 Coverage, kodun hatasız olduğunu göstermez. Sadece her satıra "uğrandığını" gösterir.
    
- **Assertion Yokluğu:** Test kodu çalıştırır ama `Assert` ile sonucu kontrol etmezse, o test "Yeşil" (Başarılı) görünür ama hiçbir şeyi test etmez. Buna "Assertion Free Testing" denir ve büyük bir aldatmacadır.
    

---

