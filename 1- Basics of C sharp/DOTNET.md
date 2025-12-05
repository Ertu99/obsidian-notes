
### 1. .NET Nedir? (Kavram Kargaşasını Çözelim)

Öncelikle şu isim karmaşasını netleştirelim. Roadmap'te ".NET Framework" yazsa da, senin odaklanacağın yer modern **.NET**.

- **.NET Framework (Eski):** Sadece Windows'ta çalışır. (Tarihe karışıyor).
    
- **.NET Core (Devrim):** Microsoft'un "artık Linux ve Mac'te de çalışacağız" dediği, sıfırdan yazılan versiyon.
    
- **.NET (Güncel - .NET 5, 6, 7, 8...):** Artık "Core" ismini attılar. Sadece ".NET" diyorlar. Hem Windows hem Linux hem Mac'te çalışır.
    

**Özet:** .NET, C# kodunun çalışmasını sağlayan devasa bir **ekosistem ve çalışma ortamıdır.**

---

### 2. Kaputun Altında Ne Var? (Architecture)

Sen C# ile kod yazdığında, bilgisayar (CPU) bunu doğrudan anlamaz. Arada bir çevirmen ve yönetici vardır. İşte .NET'in iki ana bileşeni:

#### A. CLR (Common Language Runtime) - "Motor"

Burası işin beynidir. Senin yazdığın C# kodu önce **IL (Intermediate Language)** denilen ara bir dile çevrilir. Programı çalıştırdığında CLR devreye girer ve şu hayati işleri yapar:

1. **JIT (Just-In-Time) Compilation:** Ara dili (IL), o an kullandığın makinenin (Windows, Linux vs.) anlayacağı makine diline (0 ve 1'lere) çevirir.
    
2. **Memory Management (Garbage Collection):** Bu çok önemli. C++'ta belleği sen yönetirsin, .NET'te CLR yönetir. Kullanılmayan değişkenleri RAM'den siler (Çöpçü).
    
3. **Exception Handling:** Hata yönetim mekanizmasını sağlar.
    

#### B. BCL (Base Class Library) - "Alet Çantası"

Attığın metinde "large library of pre-built functionality" denen kısım burası. Tekerleği yeniden icat etmemen için Microsoft'un sana verdiği hazır kütüphanelerdir.

- Dosya okuma/yazma (`System.IO`)
    
- Veri listeleri (`System.Collections.Generic` -> `List<T>`)
    
- İnternete çıkma (`System.Net.Http`)
    

---

### 3. Pratik: Bu "Runtime"ı Görelim

Teoriyi konuştuk, şimdi bu anlattıklarımın kodda nerede olduğunu görelim. Basit bir konsol uygulamasında CLR'ın varlığını ve BCL'i nasıl kullandığımızı kanıtlayalım.

C#

```
using System;
using System.Runtime.InteropServices; // BCL'den bir kütüphane çağırdık.

namespace DotNetBasics
{
    class Program
    {
        static void Main(string[] args)
        {
            // 1. BCL KULLANIMI:
            // "Console" sınıfı ve "WriteLine" metodu BCL'in bir parçasıdır.
            // Biz yazmadık, .NET bize hazır sundu.
            Console.WriteLine("Merhaba .NET Dünyası!");

            // 2. CLR (RUNTIME) BİLGİLERİ:
            // Kodumuz şu an hangi işletim sisteminde ve hangi framework sürümünde çalışıyor?
            // Bunu bize Runtime (CLR) söyler.
            
            Console.WriteLine($"işletim Sistemi: {RuntimeInformation.OSDescription}");
            Console.WriteLine($"Framework Sürümü: {RuntimeInformation.FrameworkDescription}");
            Console.WriteLine($"İşlemci Mimarisi: {RuntimeInformation.ProcessArchitecture}");

            // 3. GARBAGE COLLECTION (GC) KANITI:
            // Normalde bunu çağırmayız ama CLR'ın bellek yönetimini tetikleyebiliriz.
            long memoryUsed = GC.GetTotalMemory(false);
            Console.WriteLine($"Şu an kullanılan bellek: {memoryUsed} bytes");
        }
    }
}
```

Bu kodu çalıştırdığında, hangi işletim sistemindeysen (Windows, Linux, Mac) ona uygun çıktıyı dinamik olarak alırsın. İşte bu **CLR**'ın ve **Cross-Platform** özelliğinin gücüdür.

---

`Build` ettiğinde oluşan `.dll` veya `.exe` dosyasının içinde **Makine Kodu (0 ve 1) YOKTUR.** Onun yerine **IL (Intermediate Language - Ara Dil)** dediğimiz, insan diline biraz daha yakın ama bilgisayarın henüz anlamadığı özel bir kod vardır.

Süreç şöyle işler:

1. **Derleme (Build Anı):** Sen kodu yazarsın, derleyiciler (Compiler) bunu **IL**'e çevirip `.dll` dosyasına koyar. (Henüz makine kodu yok!)
    
2. **Çalıştırma (Run Anı):** Sen programa çift tıkladığında **CLR** devreye girer. O `.dll` içindeki IL kodunu alır ve **o an (Just-In-Time)** makine koduna (0-1) çevirip işlemciye verir.
    

**Neden Böyle?** Eğer `.dll` içinde direkt makine kodu olsaydı, Windows'ta derlediğin proje Linux'ta çalışmazdı. İçinde "Ara Dil (IL)" olduğu için, Linux'taki CLR alıp Linux'a göre, Windows'taki CLR alıp Windows'a göre çeviriyor.