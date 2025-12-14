Güzel, 3. Part olan **.NET CLI** (Command Line Interface), senin Pop!_OS (Linux) kullandığın için zaten doğal ortamın sayılır.

İşletim sisteminden bağımsız olarak, bir .NET geliştiricisinin elinin altında sürekli duran "İsviçre Çakısı" budur. Rider veya Visual Studio gibi IDE'ler de arka planda aslında bu komutları çalıştırır.

Hadi terminali aç ve .NET dünyasının kontrol merkezine girelim.

---

### 1. .NET CLI Nedir?

C# kodunu derlemek, paket yüklemek, projeyi ayağa kaldırmak veya yayınlamak için kullandığımız **komut satırı aracıdır.**

- **Neden Önemli?** Sunucularda (Deploy ettiğinde) Rider veya Visual Studio arayüzü olmayacak. Sadece siyah ekran (terminal) ve bu komutlar olacak.
    
- **Linux Dostu:** Windows'ta ne yazıyorsan, Pop!_OS terminalinde de aynısını yazacaksın.
    

---

### 2. En Sık Kullanacağın 5 Komut (Master Bunları Bilmeli)

Senin için en kritik komutları, gerçek bir senaryo üzerinden sırasıyla anlatıyorum. Terminalini açıp deneyebilirsin.

#### A. `dotnet new` (Yaratma)

Yeni bir proje şablonu oluşturur. Sıfırdan kod yazmak yerine iskeleti senin için kurar.

- **Kullanım:** `dotnet new <şablon_türü> -n <ProjeAdı>`
    
- **Örnekler:**
    
    - `dotnet new console` (Basit konsol uygulaması)
        
    - `dotnet new webapi` (API projesi - senin hedefin)
        
    - `dotnet new sln` (Boş bir Solution dosyası)
        

#### B. `dotnet restore` (Tamir/Hazırlık)

Projenin ihtiyaç duyduğu kütüphaneleri (NuGet paketlerini) internetten indirip yerine koyar.

- _Not: Artık `build` ve `run` komutları bunu otomatik yapıyor ama hata alırsan "bir restore at" lafını çok duyarsın._
    

#### C. `dotnet build` (İnşa Etme)

Yazdığın C# kodunu, CLR'ın anlayacağı o ara dile (IL - dll dosyasına) çevirir. Kodda hata var mı yok mu burada görürsün.

- **Çıktı Yeri:** Genelde `bin/Debug/net8.0/` klasörüne dosya oluşturur.
    

#### D. `dotnet run` (Çalıştırma)

Projeyi derler (`build`) ve hemen ardından çalıştırır. Geliştirme yaparken en çok buna basacaksın.

- **Kullanım:** Proje klasörünün içindeyken sadece `dotnet run` yazman yeterli.
    

#### E. `dotnet add package` (Eklenti Kurma)

Projeye dışarıdan bir kütüphane ekler. İleride Dapper, RabbitMQ veya Redis kullanırken bunu çok kullanacaksın.

- **Örnek:** `dotnet add package Dapper`
    

---

`dotnet run` komutu aslında şuna eşittir: `dotnet build` + (Eğer başarılıysa) -> `Programı Çalıştır`

Kodunda hata varsa derleyici (compiler) o hatayı yakalar, "Ben bunu Ara Dile (IL) çeviremem, ne dediğini anlamadım" der ve işlemi durdurur. Bu yüzden `run` aşamasına hiç geçemez.