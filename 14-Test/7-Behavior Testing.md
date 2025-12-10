
Şimdi Testing dünyasının son ve en sosyal katmanına, **Behavior Driven Development (BDD)** felsefesine ve onun .NET dünyasındaki temsilcisi **SpecFlow**'a geliyoruz.

Unit ve Integration testleri "Kodun doğru çalışıyor mu?" sorusunu cevaplar.

SpecFlow ise "Doğru kodu mu yazıyoruz?" sorusunu cevaplar.

Yazılımcı ile İş Birimi (Analist/Müşteri) arasındaki dil bariyerini yıkan bu aracı; **Gherkin Dili**, **Context Injection (Veri Taşıma)** ve **Living Documentation** kavramları üzerinden mühendislik derinliğinde inceleyelim.

---

### 1. Felsefe: The "Lost in Translation" Problem

Bir proje neden başarısız olur? Genelde kodun kalitesinden değil, **yanlış anlaşılmadan** dolayı.

- **Müşteri:** "Sepette ürün varsa indirim yap." (Hangi ürün? Ne kadar indirim?)
    
- **Yazılımcı:** `if (cart.Count > 0) discount = 10;` (Kafasına göre yazar).
    

SpecFlow (BDD) Çözümü:

Herkesin anlayabileceği ortak bir dil (Ubiquitous Language) kullanılır. Buna Gherkin denir.

Kod yazılmadan önce senaryo yazılır.

---

### 2. Mimari: Feature File & Step Definitions

SpecFlow projesi iki ana bacaktan oluşur:

#### A. Feature File (.feature)

Düz metin dosyasıdır. Visual Studio'da renklendirilmiş olarak görünür.

Gherkin

```
Feature: Sepet İndirimleri
  Müşterileri alışverişe teşvik etmek için indirimler uygulanmalı.

  Scenario: 100 TL üzeri alışverişte %10 indirim
    Given Kullanıcı sepetine 150 TL değerinde ürün eklemiştir
    When Sepeti onayladığında
    Then Toplam tutar 135 TL olmalıdır
```

#### B. Step Definitions (.cs)

SpecFlow, yukarıdaki her satırı (Step) bir C# metoduna bağlar (Binding). Regex (Düzenli İfade) kullanarak eşleştirme yapar.

C#

```cs
[Binding]
public class SepetSteps
{
    private Cart _cart;
    private decimal _finalPrice;

    [Given(@"Kullanıcı sepetine (.*) TL değerinde ürün eklemiştir")]
    public void GivenUrunEkle(int fiyat)
    {
        _cart = new Cart();
        _cart.AddProduct(new Product { Price = fiyat });
    }

    [When(@"Sepeti onayladığında")]
    public void WhenSepetiOnayla()
    {
        _finalPrice = _cart.Checkout();
    }

    [Then(@"Toplam tutar (.*) TL olmalıdır")]
    public void ThenTutarKontrol(int beklenenTutar)
    {
        Assert.Equal(beklenenTutar, _finalPrice);
    }
}
```

---

### 3. Mühendislik Harikası: Scenario Outline (Data Driven BDD)

Aynı mantığı 10 farklı değer için test etmek istersen, 10 tane Scenario yazmazsın. **Scenario Outline** kullanırsın. (xUnit `[Theory]` mantığı).

Gherkin

```
Scenario Outline: Farklı tutarlar için indirim kontrolü
  Given Sepet tutarı <Tutar> TL ise
  When İndirim hesaplandığında
  Then Sonuç <Sonuc> TL olmalıdır

Examples:
  | Tutar | Sonuc |
  | 100   | 90    |
  | 200   | 180   |
  | 50    | 50    |
```

SpecFlow bunu arka planda xUnit `[InlineData]` attribute'larına dönüştürür ve 3 ayrı test çalıştırır.

---

### 4. En Kritik Konu: State Management (Context Injection)

SpecFlow adımları (Given, When, Then) farklı C# metotlarıdır. Hatta farklı sınıflarda bile olabilirler.

Soru: Given adımında oluşturduğum _cart nesnesini, When adımına nasıl taşıyacağım?

Yöntem A: Private Field (Basit ama Kısıtlı)

Eğer tüm adımlar aynı Steps.cs sınıfındaysa private Cart _cart; alanı iş görür. Ama proje büyüyünce adımlar farklı dosyalara bölünür.

Yöntem B: Context Injection (Mühendislik Çözümü)

SpecFlow, kendi içinde mini bir Dependency Injection (DI) mekanizmasına sahiptir.

Veriyi taşımak için bir POCO (Plain Old CLR Object) sınıfı oluşturursun.

C#

```cs
// 1. Veri Taşıyıcı
public class CartContext {
    public Cart Cart { get; set; }
    public decimal Result { get; set; }
}

// 2. Step Sınıfları (Constructor Injection)
[Binding]
public class GivenSteps {
    private readonly CartContext _context;
    public GivenSteps(CartContext context) { _context = context; } // SpecFlow otomatik doldurur

    [Given(...)]
    public void Setup() { _context.Cart = new Cart(); }
}

[Binding]
public class ThenSteps {
    private readonly CartContext _context;
    public ThenSteps(CartContext context) { _context = context; } // AYNI nesne gelir!

    [Then(...)]
    public void Verify() { Assert.NotNull(_context.Cart); }
}
```

SpecFlow, bir senaryo boyunca `CartContext` nesnesini **Singleton** gibi yaşatır ve tüm step sınıflarına aynı örneği enjekte eder. Senaryo bitince yok eder.

---

### 5. Hooks (Kancalar)

Testlerden önce veya sonra çalışacak kodlar için (Veritabanı temizleme, Browser açma vb.) **Hooks** kullanılır.

- `[BeforeScenario]`: Her senaryodan önce çalışır.
    
- `[AfterStep]`: Her adımdan sonra çalışır (Hata varsa ekran görüntüsü almak için idealdir).
    
- `[BeforeFeature]`: Dosya başında bir kere çalışır.
    

---

### 6. Living Documentation (Yaşayan Doküman)

SpecFlow'un en büyük satış noktasıdır.

SpecFlow.Plus.LivingDoc aracı, test sonuçlarını ve .feature dosyalarını birleştirip harika bir HTML Raporu oluşturur.

- Bu raporu müşteriye/analiste gönderirsin.
    
- Müşteri: "Evet, benim istediğim kurallar (Feature File) bunlardı ve hepsi Yeşil (Geçmiş). Demek ki kod doğru." der.
    

---
