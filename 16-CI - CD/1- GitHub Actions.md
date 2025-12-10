Şimdi CI/CD dünyasının yükselen yıldızı, "Kodun olduğu yerde otomasyonu da yapalım" felsefesinin ürünü **GitHub Actions** konusuna geçiyoruz.

Jenkins veya TeamCity gibi harici sunucular kurma devrini kapatan bu teknolojiyi; **Workflow Anatomisi**, **Matrix Stratejileri**, **Artifact Yönetimi** ve **Service Containers** gibi en derin mühendislik detaylarıyla, tek seferde ve eksiksiz inceliyoruz.

---

### BÖLÜM 1: GİRİŞ (NE VE NEDEN?)

#### "Kod Deposu"ndan "Kod Fabrikası"na

Eskiden GitHub sadece kodlarımızı sakladığımız bir arşivdi (Kod için Dropbox). Kodunu oraya atardın, sonra başka bir program (Jenkins/Azure DevOps) o kodu oradan çeker, derler ve sunucuya atardı.

**GitHub Actions**, bu aracıları ortadan kaldırdı. GitHub artık sadece bir depo değil, içinde robotların çalıştığı bir fabrikadır.

- Sen kodu "Push" ettiğin anda, GitHub'ın kendi sunucularında (Runner) sanal bir bilgisayar açılır.
    
- Senin verdiğin emir listesine (YAML) göre kodu derler, test eder, paketler ve buluta yollar.
    
- İş bitince o sanal bilgisayar yok edilir.
    

**Neden Kullanıyoruz?**

1. **Sıfır Kurulum:** Sunucu kurmak, bakım yapmak yok.
    
2. **Kod ile Yan Yana:** CI/CD süreçlerin (YAML dosyaları) projenin kodlarıyla aynı klasörde (`.github/workflows`) durur. Versiyonlanabilir.
    
3. **Marketplace:** Binlerce hazır aksiyon (Docker Login, Azure Deploy, Slack Notify) vardır. Tek satırla eklersin.
    

---

### BÖLÜM 2: MİMARİ VE KAVRAMLAR

GitHub Actions hiyerarşisi 4 katmandan oluşur. Bu yapıyı anlamadan YAML yazamazsın.

1. **Workflow (İş Akışı):** Sürecin tamamıdır. "Proje Derleme Süreci" gibi. Bir YAML dosyası bir Workflow'dur.
    
2. **Event (Tetikleyici):** Workflow ne zaman başlayacak?
    
    - `on: push` (Kod gelince)
        
    - `on: schedule` (Her gece 03:00'te - CRON)
        
    - `on: workflow_dispatch` (Ben butona basınca - Manuel)
        
3. **Job (Görev):** Bir sanal makinede (Runner) çalışan iş grubudur.
    
    - _Önemli:_ Varsayılan olarak Job'lar **paralel** çalışır. "Build" ve "Test" aynı anda başlar. Eğer sıralı olmasını istersen `needs: build` diyerek bağımlılık belirtmelisin.
        
4. **Step (Adım):** Job'ın içindeki tekil komutlardır.
    
    - `Uses:` Hazır bir Action kullan (örn: `actions/checkout`).
        
    - `Run:` Terminal komutu çalıştır (örn: `dotnet build`).
        

---

### BÖLÜM 3: .NET İÇİN ÖRNEK PRODUCTION PIPELINE

Sadece "Hello World" değil, gerçek bir .NET projesinde kullanacağın yapı şöyledir:

YAML

```yaml
name: Production Pipeline

on:
  push:
    branches: [ "main" ] # Sadece main branch

jobs:
  build-and-test:
    runs-on: ubuntu-latest # Linux Runner
    
    steps:
    # 1. Kodu Çek
    - name: Checkout Code
      uses: actions/checkout@v3

    # 2. .NET Kurulumu ve Önbellek (Hızlandırıcı)
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '7.0.x'
        cache: true          # NuGet paketlerini otomatik cache'le!
        cache-dependency-path: '**/packages.lock.json'

    # 3. Bağımlılıkları Yükle
    - name: Restore
      run: dotnet restore

    # 4. Derle (Build)
    - name: Build
      run: dotnet build --no-restore --configuration Release

    # 5. Test Et
    - name: Test
      run: dotnet test --no-build --verbosity normal

    # 6. Yayınla (Publish)
    - name: Publish
      run: dotnet publish -c Release -o ./publish_output

    # 7. Artifact (Dosya) Sakla - Sonraki Job için
    - name: Upload Artifact
      uses: actions/upload-artifact@v3
      with:
        name: my-app-files
        path: ./publish_output
        retention-days: 1 # 1 gün sakla
```

---

### BÖLÜM 4: MÜHENDİSLİK DETAYLARI VE STRATEJİLER

#### A. Artifact Management (Joblar Arası Veri Taşıma)

Her Job taze bir sanal makinede başlar.

- **Job A (Build):** Kodu derledi, `app.dll` oluşturdu. Job bitti, makine silindi.
    
- **Job B (Deploy):** Başladı. Ama `app.dll` yok!
    
- **Çözüm:** `upload-artifact` ve `download-artifact` aksiyonları.
    
    - Job A dosyayı GitHub'ın geçici deposuna yükler.
        
    - Job B oradan indirir.
        

#### B. Matrix Strategy (Çoklu Test)

Kütüphanen hem .NET 6, hem .NET 7, hem de .NET 8 üzerinde çalışmalı mı? 3 ayrı Job yazmana gerek yok.

YAML

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dotnet-version: [ '6.0.x', '7.0.x', '8.0.x' ]
    steps:
    - uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ matrix.dotnet-version }}
    - run: dotnet test
```

GitHub bunu görünce **3 tane paralel Job** başlatır. Her biri farklı versiyonla test eder. Biri bile patlarsa süreç durur.

#### C. Service Containers (Entegrasyon Testi Ortamı)

Testlerin gerçek bir Redis veya SQL Server gerektiriyorsa ne yapacaksın? GitHub Actions, Job içinde geçici **Docker Konteynerleri** açabilir.

YAML

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      # Test için geçici Redis
      redis:
        image: redis
        ports:
          - 6379:6379
    steps:
    - run: dotnet test # Kod localhost:6379'a bağlanıp test edebilir!
```

---

### BÖLÜM 5: GÜVENLİK VE SECRET YÖNETİMİ

YAML dosyaları herkese açıktır. Veritabanı şifresini buraya yazarsan hacklenirsin.

1. **Secrets:** Repo ayarlarından `Settings > Secrets and variables > Actions` kısmına girip `DB_PASSWORD` olarak kaydedersin.
    
2. **Kullanım:** YAML içinde `${{ secrets.DB_PASSWORD }}` diyerek çağırırsın.
    
3. **Log Masking:** GitHub, loglarda bu şifreyi görürse otomatik olarak `***` şeklinde sansürler.
    

---

### BÖLÜM 6: ENVIRONMENTS VE ONAY MEKANİZMASI

Canlıya (Production) çıkarken "Bir insan kontrol etsin" isteyebilirsin.

1. GitHub Repo ayarlarında **"Production"** adında bir Environment oluştur.
    
2. **"Required Reviewers"** (Zorunlu Onaylayıcılar) kısmına Takım Liderini ekle.
    
3. YAML dosyasında Deploy job'ına `environment: Production` ekle.
    

**Sonuç:** Build biter, Test biter. Deploy aşamasına gelince Pipeline **DURUR** ve Takım Liderine bildirim gider: _"Production'a çıkmak için onayınız bekleniyor."_ Onay verilirse devam eder.

---

### BÖLÜM 7: REUSABLE WORKFLOWS (DRY)

50 tane mikroservisin var. Hepsi için aynı YAML dosyasını 50 kere kopyalamak ameleliktir ve yönetilemez. GitHub Actions **Reusable Workflows** (Yeniden Kullanılabilir Akışlar) sunar.

1. `build-template.yml` adında genel bir dosya yaparsın.
    
2. Diğer projelerde `uses: ./.github/workflows/build-template.yml` diyerek bu şablonu çağırırsın.
    
3. Şablonu güncellediğinde 50 proje birden güncellenir.