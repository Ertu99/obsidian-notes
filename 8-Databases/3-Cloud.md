
Çoğu kişi Cloud'u "Bilgisayarın internette olması" zanneder. Bu, "Bulut başkasının bilgisayarıdır" (There is no cloud, it's just someone else's computer) geyiğinden ibarettir.

Ama bir **Mimar** için Cloud, **"Programlanabilir Altyapı" (Infrastructure as Code)** ve **"Sınırsız Ölçeklenebilirlik"** demektir. Eskiden sunucu almak için dilekçe yazıp 3 ay beklerdik; şimdi bir API çağrısı ile 1000 sunucuyu 1 dakikada ayağa kaldırıyoruz.

Bu konuyu, özellikle .NET ekosisteminin kralı olan **Azure** ve genel bulut mimarisi kavramları (IaaS, PaaS, Serverless) üzerinden, mühendislik derinliğinde inceleyelim.

---

### 1. Hizmet Modelleri (Sorumluluk Kimde?)

Buluta geçerken vermen gereken ilk karar: "Sunucunun ne kadarını ben yöneteceğim?"

Buna Shared Responsibility Model (Paylaşılan Sorumluluk Modeli) denir.

#### A. IaaS (Infrastructure as a Service) - "Altyapı"

- **Analoji:** Boş bir arsa kiralamak.
    
- **Nedir:** Sana boş bir Sanal Makine (VM) verirler. İçine Windows kurmak, IIS ayarlarını yapmak, güvenlik güncellemelerini takip etmek **senin** sorumluluğundadır.
    
- **Örnek:** Azure VM, AWS EC2.
    
- **Mühendislik Yükü:** En yüksektir. "Disk doldu", "Windows çöktü" dertleri senindir.
    
- **Kullanım:** Çok özel işletim sistemi ayarları gereken (Legacy) uygulamalar için.
    

#### B. PaaS (Platform as a Service) - "Platform" - **.NET İçin Standart**

- **Analoji:** Otel odası kiralamak. Elektrik, su, temizlik otelin işidir; sen sadece yaşamaya bakarsın.
    
- **Nedir:** Sana "Çalışan bir .NET Runtime ortamı" verirler. İşletim sistemiyle uğraşmazsın. Kodunu (`publish`) atarsın ve çalışır.
    
- **Örnek:** **Azure App Service**, AWS Elastic Beanstalk.
    
- **Mühendislik Yükü:** Düşüktür. OS güncellemelerini Microsoft yapar.
    
- **Kullanım:** Modern Web API ve MVC projelerinin %90'ı buradadır.
    

#### C. FaaS (Function as a Service / Serverless) - "Sunucusuz"

- **Analoji:** Taksi tutmak. Araba kimin, bakımı ne zaman yapıldı umrunda değil. Sadece A'dan B'ye gitmek için para ödersin.
    
- **Nedir:** Sadece tek bir fonksiyon (`public void Run()`) yazarsın. Sunucu kavramı tamamen soyutlanmıştır. Kodun sadece çalıştığı saniye kadar para yazar.
    
- **Örnek:** **Azure Functions**, AWS Lambda.
    
- **Kullanım:** Arka plan işleri, resim küçültme, olay tabanlı (Event-Driven) işler.
    

---

### 2. Ölçeklenme Mühendisliği (Scaling)

Cloud'un en büyük vaadi budur. Trafik arttığında ne yapacağız?

#### A. Vertical Scaling (Scale Up - Dikey)

- **Mantık:** Sunucunun donanımını güçlendirmek.
    
- **İşlem:** 4 GB RAM'li sunucuyu kapatıp, 16 GB RAM'li sunucuya taşımak.
    
- **Sorun:** Bir sınırı vardır (Dünyanın en güçlü bilgisayarının bile limiti bellidir) ve işlem sırasında kesinti (Downtime) olabilir.
    

#### B. Horizontal Scaling (Scale Out - Yatay) - **Cloud Native Yöntem**

- **Mantık:** Sunucuyu güçlendirmek yerine, yanına aynısından 5 tane daha koymak.
    
- **İşlem:** Önüne bir **Load Balancer (Yük Dengeleyici)** koyarsın. Gelen trafiği 5 sunucuya dağıtır.
    
- **Avantaj:** Sınırı yoktur. 1000 sunucuya kadar çıkabilirsin. Kesinti olmaz.
    

---

### 3. "Stateless" (Durumsuzluk) Kuralı

Bir önceki caching dersinde "Sticky Session" riskinden bahsetmiştik. Cloud'da **Scale Out** (Yatay Büyüme) yapabilmek için uygulamanın **Stateless** olması zorunludur.

**Mühendislik Felaketi:**

- Uygulaman dosya yükleme (Upload) yapıyor.
    
- Kullanıcı profil resmini yükledi ve dosya `C:\inetpub\wwwroot\uploads` klasörüne (Sunucu A'nın diski) kaydedildi.
    
- Sistem trafiği gördü ve otomatik olarak **Sunucu B**'yi devreye aldı (Auto-Scaling).
    
- Kullanıcı F5'e bastı, Load Balancer onu Sunucu B'ye gönderdi.
    
- **Sonuç:** Kullanıcı resmini göremez (404 Not Found). Çünkü resim Sunucu A'nın diskinde kaldı. Sunucu B'nin diski boştur.
    

**Çözüm:** Cloud uygulamalarında yerel disk veya yerel RAM kullanılmaz.

- Dosyalar -> **Azure Blob Storage** / AWS S3 (Ortak Depolama).
    
- Session/Cache -> **Redis** (Ortak Önbellek).
    
- Loglar -> **Elasticsearch** / Application Insights (Ortak Loglama).
    

---

### 4. Vendor Lock-in (Tedarikçi Kilidi)

Mimar olarak vermen gereken stratejik bir karardır.

Cloud sağlayıcısının (Azure) o kadar güzel, o kadar kolay hizmetleri vardır ki (örneğin CosmosDB, Azure Service Bus), bunları kullanırsan kodun Azure'a göbekten bağlanır.

Yarın patron "AWS daha ucuz, oraya taşıyalım" dediğinde taşıyamazsın. Kodun her yerini değiştirmen gerekir.

- **Çözüm:** Dependency Injection ve Abstraction.
    
    - Kodun içinde doğrudan `AzureServiceBusSDK` kullanma.
        
    - `IMessageBus` diye arayüz yaz. Azure uygulamasını (`AzureBusAdapter`) ayrı bir yerde yap.
        
    - Taşınırken sadece o adaptörü değiştirirsin.
        

---

### 5. Infrastructure as Code (IaC)

Cloud'da sunucular "Evcil Hayvan" (Pet) değil, "Büyükbaş Hayvan" (Cattle) gibidir. İsim vermeyiz, hastalanınca iyileştirmeye çalışmayız; yenisini açarız.

Bu yüzden Azure Portal'a girip elle "tık tık" sunucu kurulmaz. Altyapı kod ile yazılır.

- **Araçlar:** **Terraform**, **Pulumi**, **Azure Bicep**.
    
- **Fayda:** `terraform apply` dersin, kodun gider sıfırdan tüm mimariyi (Veritabanı, Redis, API Sunucuları, Ağ ayarları) 5 dakikada kurar.
    

---
