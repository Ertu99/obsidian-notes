Unit Testing dünyasında `xUnit` ile testleri nasıl yazacağımızı, `Fixture` ile ortamı nasıl kuracağımızı öğrendik. Şimdi ise testin en önemli anına, yani **"Sonuç Doğru mu?" (Assert)** kararını verdiğimiz noktaya geliyoruz.

xUnit'in kendi `Assert` sınıfı iş görür, ancak hata mesajları genellikle "soğuk ve teknik"tir. **Shouldly**, test kodunu bir İngilizce cümle gibi okumanı sağlayan ve hata aldığında sana **"Neyin yanlış gittiğini"** çok net anlatan bir kütüphanedir.

Bu konuyu; **Okunabilirlik (Readability)**, **Hata Mesajı Kalitesi (Diagnostics)** ve **Fluent Assertion** mantığı üzerinden inceleyelim.

---

### 1. Felsefe: "Assert" vs "Should"

Geleneksel test yazımında (xUnit Assert) kod şöyledir:

C#

```cs
// xUnit
Assert.Equal(expected: 5, actual: result);
```

- **Sorun:** Parametre sırasını karıştırmak çok kolaydır (`expected` hangisi, `actual` hangisi?). Hata mesajı şöyledir: _"Assert.Equal() Failure: Expected: 5, Actual: 4"_.
    

**Shouldly Yaklaşımı:**

C#

```cs
// Shouldly
result.ShouldBe(5);
```

- **Avantaj:** Kod İngilizce cümle gibi okunur: _"Result should be 5"_ (Sonuç 5 olmalı). Parametre sırası derdi yoktur.
    

---

### 2. Hata Mesajlarının Gücü (The Killer Feature)

Shouldly'nin en büyük olayı, test patladığında sana verdiği mesajdır. Kodun kaynak kodunu analiz eder ve sana değişken ismini söyler.

**Senaryo:** Sepet toplamı 100 olmalıydı ama 90 çıktı.

- xUnit:
    
    Expected: 100, Actual: 90 (Hangi değişkenin 90 olduğunu söylemez).
    
- **Shouldly:**
    
    Plaintext
    
    ```
    cartTotal
        should be
    100
        but was
    90
    ```
    
    Bu mesajı okuyan bir yazılımcı, debug yapmadan hatanın `cartTotal` değişkeninde olduğunu anlar.
    

---

### 3. Koleksiyon ve String İşlemleri

Shouldly, karmaşık veri tiplerini kontrol ederken çok güçlüdür.

**Koleksiyonlar:**

C#

```cs
var users = new[] { "Ahmet", "Mehmet", "Ayşe" };

// xUnit ile bunu yazmak işkencedir
// Shouldly ile:
users.ShouldContain("Ahmet");
users.ShouldNotBeEmpty();
users.Count().ShouldBeGreaterThan(2);
```

**Stringler:**

C#

```cs
var error = "Hata: Kullanıcı bulunamadı (ID: 55)";

error.ShouldStartWith("Hata");
error.ShouldContain("bulunamadı");
error.ShouldMatch(@"ID: \d+"); // Regex desteği
```

---

### 4. Exception (Hata) Testleri

Bir metodun hata fırlatıp fırlatmadığını test etmek `try-catch` ile yapılmaz. Shouldly bunu çok şık bir hale getirir.

C#

```cs
// Bu metot çağrıldığında "ArgumentNullException" fırlatmalı
var exception = Should.Throw<ArgumentNullException>(() => _service.GetUser(null));

// Hata mesajını da kontrol edelim
exception.Message.ShouldContain("username cannot be null");
```

---

### 5. FluentAssertions vs Shouldly

Sektörde iki büyük rakip vardır.

1. **FluentAssertions:** Çok daha fazla özellik ve detay sunar (`.Should().Be().And.HaveCount()...`). Zincirleme metotları çok uzundur.
    
2. **Shouldly:** Daha sadedir (`.ShouldBe()`). Hata mesajları daha "akıllı"dır.
    

**Mühendislik Kararı:** Eğer amacın sadece okunabilirlik ve net hata mesajı ise **Shouldly**. Eğer çok karmaşık nesne graflarını (Nested Objects) derinlemesine karşılaştırmak istiyorsan **FluentAssertions** seçebilirsin.

---
