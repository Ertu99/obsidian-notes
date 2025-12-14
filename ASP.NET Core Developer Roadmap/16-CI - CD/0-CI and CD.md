### BÖLÜM 1: GİRİŞ (NE VE NEDEN?)

#### Fabrika Analojisi

Eskiden (CI/CD yokken), yazılım geliştirmek bir **"El Sanatları Atölyesi"** gibiydi. Sen kodunu yazardın (parçayı üretirdin). Sonra bu kodu bir USB'ye atar veya FTP ile sunucuya kopyalardın (elle taşıma). Sunucuya bağlanıp "Derle" tuşuna basardın (elle montaj). Sonra "Test et" derdin. Hata çıkarsa her şeyi geri alırdın.

- **Sorun:** Çok yavaştı, hata yapma riski çok yüksekti ("Ah, config dosyasını kopyalamayı unuttum!") ve herkes birbirini beklerdi.
    

**CI/CD ise modern bir "Otomobil Fabrikası"dır.** Sen sadece **"Kodu Gönder" (Commit)** butonuna basarsın. Geri kalan her şey otomatik bir bantta ilerler:

1. Robotlar kodu alır, derler (Build).
    
2. Robotlar kodu test eder (Test).
    
3. Robotlar kodu paketler (Package).
    
4. Robotlar kodu canlı sunucuya taşır (Deploy).
    

Sen kahveni içerken, sistem senin kodunu 5 dakika içinde müşterinin önüne koyar.

#### Kavramlar: CI ve CD Nedir?

- **CI (Continuous Integration - Sürekli Entegrasyon):** "Kodu birleştirme ve test etme" aşamasıdır.
    
    - _Amaç:_ "Benim yazdığım kod, Ahmet'in yazdığı kodu bozdu mu?" sorusunu her dakika sormaktır.
        
- **CD (Continuous Delivery/Deployment - Sürekli Teslimat/Dağıtım):**
    
    - _Delivery:_ Kodu paketleyip "Deploy etmeye hazır" hale getirmektir (Son tuşa insan basar).
        
    - _Deployment:_ İnsan müdahalesi olmadan kodu direkt canlıya (Production) almaktır (Cesaret ister!).
        

---

### BÖLÜM 2: MÜHENDİSLİK DERİNLİĞİ (PIPELINE ANATOMİSİ)

Bir .NET projesi için **Production-Grade (Canlıya Hazır)** bir Pipeline nasıl olmalıdır? GitHub Actions veya Azure DevOps üzerinde kuracağın yapı adım adım şöyledir:

#### 1. Stage: Build (İnşaat)

Bu aşamanın amacı, kodun derlenebilir olduğunu kanıtlamaktır.

- **Restore:** `dotnet restore` (NuGet paketlerini indir).
    
- **Build:** `dotnet build -c Release --no-restore` (Kodu derle).
    
- **Mühendislik Detayı (Cache):** Her seferinde 500 MB NuGet paketi indirmemek için `Cache` mekanizması kullanılır. Pipeline, önceki build'den kalan paketleri hatırlar.
    

#### 2. Stage: Test & Quality (Kalite Kontrol)

Sadece derlenmesi yetmez, çalışması lazım.

- **Unit Tests:** `dotnet test`. Eğer tek bir test bile kırmızı yanarsa (Fail), bant durur! Kod asla canlıya gitmez.
    
- **Code Analysis (SonarQube):** Kod kalitesi kontrol edilir. "Çok fazla iç içe IF kullanmışsın", "Şifreyi koda gömmüşsün" gibi hataları yakalar.
    
- **Vulnerability Scan:** Kullandığın kütüphanelerde güvenlik açığı var mı diye bakar (Örn: `dotnet list package --vulnerable`).
    

#### 3. Stage: Packaging (Paketleme)

Kod testten geçti. Şimdi onu taşınabilir bir kutuya koymalıyız.

- **Artifact:** `dotnet publish -o /app` komutuyla çıkan DLL ve EXE dosyalarıdır.
    
- **Docker Image:** Eğer konteyner kullanıyorsan, bu aşamada `docker build` yapılır ve oluşan imaj bir **Container Registry**'ye (Docker Hub, Azure ACR) gönderilir (`docker push`).
    

#### 4. Stage: Deployment (Dağıtım)

Paketin hedef sunucuya gönderilmesi.

- **Environment Strategy:** Kod önce **DEV** ortamına, sonra **TEST (QA)** ortamına, en son onay gelirse **PROD** ortamına gider.
    
- **Variable Substitution:** En kritik kısımdır.
    
    - Kodun içinde veritabanı şifresi yoktur.
        
    - Pipeline, kodu sunucuya atarken `appsettings.json` içindeki boş alanları o ortama uygun şifrelerle doldurur. (Dev DB şifresi ayrı, Prod DB şifresi ayrı).
        

---

### BÖLÜM 3: GÜVENLİK VE BEST PRACTICES (SECRET MANAGEMENT)

Bir CI/CD sürecinde yapılan en büyük hata, şifreleri (API Key, Connection String) kodun içine veya YAML dosyasına açık açık yazmaktır.

- **Secrets (Sırlar):** GitHub veya Azure DevOps'un "Secrets" kasası vardır. Şifreleri oraya kaydedersin.
    
- **Injection:** Pipeline çalışırken bu şifreleri ortam değişkeni (Environment Variable) olarak sunucuya enjekte eder. Loglarda bu şifreler `***` olarak görünür.
    

---

### BÖLÜM 4: GELİŞMİŞ DAĞITIM STRATEJİLERİ (DEPLOYMENT STRATEGIES)

Uygulamayı canlıya alırken "Sistemi kapatıp yeni versiyonu yüklemek" (Downtime) amatörce bir iştir. Profesyonel stratejiler şunlardır:

#### A. Blue-Green Deployment

İki adet ikiz sunucun vardır: **Blue** (Şu an çalışan - Canlı) ve **Green** (Boşta).

1. Yeni versiyonu **Green** sunucusuna yüklersin.
    
2. Green'i test edersin.
    
3. Load Balancer'daki "Trafik Yönü" ibresini Blue'dan Green'e çevirirsin.
    
4. **Sonuç:** Kesinti sıfırdır. Hata varsa ibreyi hemen geri (Rollback) çevirirsin.
    

#### B. Canary Deployment (Kanarya Dağıtımı)

Maden işçileri gaz sızıntısını anlamak için madene önce kanarya kuşu indirirdi.

1. Yeni versiyonu sunucularının sadece %5'ine yüklersin.
    
2. Trafiğin %5'i (şanslı kullanıcılar) yeni versiyonu görür.
    
3. Hata oranı artıyor mu diye izlersin (Monitoring).
    
4. Sorun yoksa yavaş yavaş %25, %50, %100 yaparsın.
    

---

### BÖLÜM 5: ARAÇLAR (TOOLS)

Bir .NET geliştiricisi olarak hangi araçları bilmelisin?

1. **GitHub Actions:** Şu an dünyanın en popüler, en hızlı büyüyen aracı. GitHub'ın içinde gömülüdür. YAML ile yazılır. Küçük ve orta ölçekli projeler için standarttır.
    
2. **Azure DevOps (ADO):** Kurumsal dünyanın kralıdır. Proje yönetimi (Boards), Repo ve Pipeline tek çatı altındadır. Çok detaylı yetki yönetimi vardır.
    
3. **Jenkins:** Eski toprak. Çok güçlüdür ama bakımı zordur (Kendi sunucuna kurman gerekir). Modern dünyada yerini cloud tabanlı araçlara bırakıyor.
    

---

### BÖLÜM 6: ÖRNEK BİR GITHUB ACTIONS YAML (.NET)

İşte senin için "Kopyala-Yapıştır" yapabileceğin, CI aşamasını halleden örnek bir Pipeline dosyası:

YAML

```yaml
name: .NET Core CI

on:
  push:
    branches: [ "main" ] # Sadece main'e kod gelince çalış
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest # Linux makinede çalış

    steps:
    - uses: actions/checkout@v3 # Kodu çek
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 7.0.x # .NET 7 kur
        
    - name: Restore dependencies
      run: dotnet restore
      
    - name: Build
      run: dotnet build --no-restore --configuration Release
      
    - name: Test
      run: dotnet test --no-build --verbosity normal # Testleri koş!
```



### 7. CI/CD (GitHub Actions / Azure DevOps)

**🧒 6 Yaşındaki Çocuğa (Robotlu Pizza Dükkanı Analojisi):** "Eskiden pizzacı amca sipariş geldiğinde hamuru eliyle açar, malzemeyi koyar, fırına atardı. Bazen tuzu unuturdu, bazen pizzayı yakardı, bazen de çok yavaş olurdu. **CI/CD**, dükkana kocaman bir **Pizza Robotu** almak gibidir. Sen sadece düğmeye basarsın (**Commit**).

1. Robot hamuru hazırlar (**Build**).
    
2. Sonra küçük bir parça tadına bakar, 'Tuzu iyi mi?' diye kontrol eder (**Test**). Eğer kötüyse pizzayı çöpe atar, sana da 'Tuz koymamışsın!' diye kırmızı ışık yakar.
    
3. Tadı güzelse pizzayı kutuya koyar (**Package**).
    
4. En sonunda da pizzayı müşterinin evine drone ile gönderir (**Deploy**). Robot hiç yorulmaz, tuzu asla unutmaz ve her pizzayı aynı mükemmellikte yapar."
    

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "Modern yazılım geliştirmede, kodun kalitesini ve teslimat hızını artırmak için manuel süreçler yerine tam otomatize edilmiş **CI/CD (Sürekli Entegrasyon ve Dağıtım)** boru hatları (Pipelines) kurulması endüstri standardıdır. Bir DevOps kültürünün mimari temelleri şunlardır:

- **Quality Gates (Kalite Kapıları):** Kodun sadece derlenmesi (Build) yeterli değildir. Pipeline sürecine **Unit Testler**, **SonarQube** ile statik kod analizi ve güvenlik taramaları (Vulnerability Scan) entegre edilerek, hatalı kodun üretim ortamına ulaşması daha en baştan engellenir.
    
- **Secret Management:** Güvenlik ihlallerini önlemek için; veritabanı şifreleri veya API anahtarları asla kodun içinde (Hardcoded) tutulmaz. Azure KeyVault veya GitHub Secrets gibi güvenli kasalardan, deployment anında **Environment Variable** olarak enjekte edilir.
    
- **Zero Downtime Deployment:** Canlıya geçişlerde hizmet kesintisi yaşanmaması için **Blue-Green** veya **Canary** dağıtım stratejileri uygulanır. Bu sayede yeni versiyon önce izole bir ortamda veya kısıtlı bir kullanıcı kitlesinde doğrulanır, olası bir hatada anında geri alma (Rollback) imkanı sağlanır.
    
- **Infrastructure as Code:** Pipeline sadece uygulama kodunu değil, Terraform veya Bicep gibi araçlarla altyapıyı da yöneterek, ortamlar arası tutarlılığı (Consistency) garanti eder."