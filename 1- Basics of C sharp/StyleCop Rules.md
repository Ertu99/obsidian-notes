
### 1. StyleCop Nedir? (Kod Polisi)

Sen bir takıma girdiğinde, herkes kendi kafasına göre kod yazarsa kaos çıkar.

- Ahmet süslü parantezi `{` satırın sonuna koyar.
    
- Mehmet `{` işaretini alt satıra koyar.
    
- Ayşe değişken ismine `musteriAdi` der, Fatma `Musteri_Adi` der.
    

Kod çalışır mı? Evet, CLR için fark etmez.

Ama kodun okunabilirliği biter. StyleCop, elinde düdükle bekleyen bir hakemdir. "Kurallara uymadın, burayı düzelt!" diyerek uyarı verir.

---

### 2. Nasıl Çalışır? (Modern Yöntem)

Eskiden ayrı bir programdı ama artık `.NET Core` dünyasında bu bir **NuGet Paketi** (StyleCop.Analyzers) olarak gelir. Yani projenin içine gömülür.

Sen Pop!_OS terminalinde veya Rider'da kodu derlediğinde (`dotnet build`), StyleCop devreye girer ve kodunu tarar.

Örnek bir StyleCop kuralı (SA1200):

StyleCop der ki: "Using ifadelerini (importları) namespace'in içine yazmalısın." (Bu tartışmalı bir kuraldır ama varsayılan olarak böyledir).

**Hatalı (StyleCop Kızar):**

C#

```
using System; // HATA! Namespace'in dışında.

namespace BenimProjem
{
    class Program { ... }
}
```

**Doğru (StyleCop Mutlu):**

C#

```
namespace BenimProjem
{
    using System; // Aferin, kurala uydun.

    class Program { ... }
}
```

---

### 3. Pratik: Projeye Dahil Etmek

Az önce öğrendiğimiz CLI komutlarıyla projene StyleCop eklemek çok basittir. Terminalde proje klasöründeyken şu komutu yazarsın:

Bash

```
dotnet add package StyleCop.Analyzers
```

Bu komuttan sonra `dotnet build` yaparsan, terminalin birden sarı sarı uyarılarla (warning) dolduğunu görebilirsin. "Boşluk bırakmadın", "Yorum satırı eklemedin" gibi yüzlerce kuralı denetler.

**Not:** Kuralları `stylecop.json` veya `.editorconfig` dosyası ile özelleştirebilirsin. Mesela "Ben süslü parantezi satır sonuna koymayı seviyorum, buna kızma" diyebilirsin.

---

