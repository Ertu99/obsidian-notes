### DOCKER

Şimdi yazılım dünyasını (ve özellikle DevOps kültürünü) kökünden değiştiren, "Benim makinemde çalışıyordu" bahanesini tarihe gömen teknolojiye, **Containerization** başlığına ve onun kralı **Docker**'a geliyoruz.

Eskiden bir uygulamayı sunucuya atmak (Deployment) işkenceydi. IIS ayarları, .NET Framework versiyonu, eksik DLL'ler... Docker ile artık sunucuya kod atmıyoruz; sunucuya **"Çalışmaya Hazır Paketlenmiş Bir Evren" (Image)** atıyoruz.

Bu konuyu; **Kernel (Çekirdek) Mimarisi**, **Layer Caching (Katman Önbelleği)**, **Multi-Stage Builds** ve **Networking** üzerinden, bir Mimar derinliğinde inceleyelim.

---

### 1. Felsefe: Sanal Makine (VM) vs Konteyner

Mülakatların değişmez sorusudur: _"VMware varken neden Docker kullanayım?"_

- **Sanal Makine (VM):**
    
    - Donanımı sanallaştırır (Hypervisor).
        
    - Her VM'in içinde **tam bir İşletim Sistemi (Guest OS)** vardır (Windows/Linux).
        
    - Açılması dakikalar sürer. GB'larca yer kaplar.
        
    - _Analoji:_ Her uygulama için ayrı bir **Müstakil Ev** inşa etmek gibidir.
        
- **Konteyner (Docker):**
    
    - İşletim Sistemini sanallaştırır.
        
    - Tüm konteynerler, ana makinenin (Host) **Linux Çekirdeğini (Kernel)** ortaklaşa kullanır.
        
    - İçinde sadece uygulamanın çalışması için gereken kütüphaneler (Libs/Binaries) vardır.
        
    - Açılması milisaniyeler sürer. MB'larca yer kaplar.
        
    - _Analoji:_ Koca bir apartmanda (Host), herkese birer **Daire** (Container) vermek gibidir. Elektrik/Su tesisatı (Kernel) ortaktır.
        

---

### 2. Kaputun Altı: Namespaces ve Cgroups

Docker aslında bir "sihir" değildir. Linux çekirdeğinin yıllardır sahip olduğu iki özelliğin güzel bir paketidir:

1. **Namespaces (İzolasyon):** "Ben yalnızım."
    
    - Konteyner içindeki işlem (Process), kendini sunucudaki tek işlem sanır. Diğer konteynerleri göremez. Dosya sistemi, ağ ve kullanıcılar izoledir.
        
2. **Cgroups (Control Groups - Kaynak Yönetimi):** "Sınırların var."
    
    - "Bu konteyner en fazla 512MB RAM ve %50 CPU kullanabilir" dersin. Linux çekirdeği bunu zorlar. Bir konteynerin sunucuyu kilitlemesini engeller.
        

---

### 3. Artifact: Image vs Container

Yazılımcı olarak bu farkı Class/Object farkı gibi düşünmelisin.

- **Image (İmaj):** Class (Sınıf).
    
    - Read-Only (Sadece okunur) şablondur.
        
    - İçinde kodun, .NET Runtime ve kütüphaneler vardır.
        
    - `docker build` komutuyla oluşur.
        
- **Container (Konteyner):** Object (Nesne).
    
    - İmajın çalışan halidir (`run-time instance`).
        
    - İmajın üzerine ince bir **Yazılabilir Katman (Writable Layer)** eklenir.
        
    - `docker run` komutuyla oluşur.
        
    - Aynı İmajdan 100 tane Konteyner başlatabilirsin.
        

---

### 4. Mühendislik Harikası: Layer Caching (Katman Önbelleği)

Docker imajları tek bir blok değildir. **Katmanlardan (Layers)** oluşur. `Dockerfile` içindeki her satır yeni bir katmandır.

Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0  # Katman 1 (Base)
WORKDIR /app                           # Katman 2
COPY . .                               # Katman 3 (Kodları kopyala)
RUN dotnet restore                     # Katman 4 (Paketleri indir)
RUN dotnet publish                     # Katman 5 (Build et)
```

**Sorun:** Yukarıdaki dizilim **YANLIŞTIR.** Neden? Çünkü `COPY . .` satırında kodunun _tek bir harfi_ bile değişse, Docker o katmanın (Katman 3) değiştiğini anlar. Ve ondan sonra gelen **TÜM katmanları (4 ve 5)** baştan yapar. Yani her kod değişikliğinde `dotnet restore` çalışır ve internetten paket indirir. Build süresi uzar.

**Doğru Dizilim (Optimization):**

Dockerfile

```Dockerfile
COPY *.csproj .       # Önce SADECE proje dosyasını kopyala
RUN dotnet restore    # Paketleri indir (Cachelenir!)
COPY . .              # Sonra kalan kodları kopyala
RUN dotnet publish    # Build et
```

Artık kod değişse bile `csproj` değişmediği sürece Docker ilk adımları **Cache'ten (Önbellek)** getirir. Build süresi 2 dakikadan 5 saniyeye düşer.

---

### 5. Multi-Stage Builds (Boyut Küçültme)

Bir .NET projesini build etmek için `SDK` imajına (içinde derleyici, araçlar vs. var - 800MB) ihtiyaç vardır. Ama çalıştırmak için sadece `Runtime` imajı (80MB) yeterlidir.

Eğer SDK imajıyla production'a çıkarsan, hem disk israf edersin hem de güvenlik açığı (gereksiz araçlar) yaratırsın.

**Çözüm:** Çok aşamalı build.

1. **Stage 1 (Build):** SDK imajını kullan, kodu derle, DLL'leri üret.
    
2. **Stage 2 (Runtime):** Boş bir Runtime imajı aç.
    
3. **Copy:** Stage 1'den sadece DLL'leri al, Stage 2'ye koy.
    
4. **Sonuç:** 800MB yerine 80MB'lık tertemiz bir imaj.
    

---

### 6. Veri Kalıcılığı: Volumes

Konteynerler **Ephemeral** (Geçici) varlıklardır. Bir konteyneri silip yeniden başlattığında, içindeki "Yazılabilir Katman" silinir. Eğer veritabanını (SQL Server) konteyner içinde çalıştırıyorsan ve veriyi konteynerin içine kaydediyorsan, konteyner kapandığında **verin silinir.**

**Çözüm: Volumes** Veriyi konteynerin içinde değil, Ana Makinenin (Host) diskinde saklamalı ve konteynere bağlamalısın (Mount).

- `docker run -v /host/data:/container/data ...`
    

---

### 7. Networking (Konteynerler Nasıl Konuşur?)

Docker varsayılan olarak **Bridge Network** kullanır.

- Her konteynerin kendine özel, dışarıdan erişilemeyen bir IP'si vardır (örn: `172.17.0.2`).
    
- Konteynerler birbirini bu IP ile veya (Docker Compose kullanıyorsan) **Servis Adı** ile bulur.
    
    - Web API konteyneri, SQL konteynerine bağlanırken `Server=localhost` yazamaz! (Localhost konteynerin kendisidir).
        
    - `Server=db_container` yazmalıdır.


### KUBERNETES

Kubernetes (kısaca K8s), modern bulut dünyasının işletim sistemidir. Konuyu; **Mimari Bileşenler**, **Networking**, **Depolama (Storage)** ve **Gelişmiş Dağıtım Stratejileri** üzerinden, eksiksiz bir mühendislik derinliğinde inceleyelim.

---

### 1. Giriş: Neden Kubernetes? (Liman Analojisi)

Docker ile uygulamanı bir "Konteyner"e (Yük Konteyneri) paketledin. Bu harika. Ama elinde 1 tane değil, **1.000 tane konteyner** varsa işler karışır.

- Bu 1.000 konteyneri hangi gemilere (Sunuculara) yükleyeceksin?
    
- Gemilerden biri batarsa (Sunucu çökerse), içindeki konteynerleri kim kurtarıp başka gemiye taşıyacak?
    
- Yük artarsa (Trafik), kim yeni konteynerler getirecek?
    

İşte **Kubernetes**, bu limanı yöneten **"Otomasyon Sistemi"**dir. İnsan müdahalesi olmadan; konteynerleri sunuculara dağıtır, öleni yeniden başlatır, trafiği yönlendirir ve sistemi ayakta tutar.

---

### 2. Mimari: Beyin ve Kaslar (Master & Worker Nodes)

Kubernetes tek bir program değil, bir kümeler (Cluster) bütünüdür. İki ana rolden oluşur:

#### A. Control Plane (Master Node) - BEYİN

Sistemin kararlarını veren merkezdir. Uygulamalar burada çalışmaz, burada sadece "yönetim" yapılır.

1. **API Server:** Sistemin kapısıdır. Sen (`kubectl`) veya diğer parçalar sadece burayla konuşur.
    
2. **Scheduler (Planlamacı):** "Yeni bir Pod geldi, hangi sunucuda boş yer var?" diye bakar ve atama yapar.
    
3. **Controller Manager:** Sistemin bekçisidir. "3 tane Pod olsun dedim, şu an 2 tane var. Hemen 1 tane daha yaratayım" diyen döngüdür.
    
4. **Etcd:** Sistemin hafızasıdır (Key-Value Store). Cluster'ın o anki durumu (State) burada tutulur.
    

#### B. Worker Nodes (İşçi Düğümleri) - KASLAR

Uygulamalarının (Pod'ların) gerçekten çalıştığı sunuculardır.

1. **Kubelet:** Geminin kaptanıdır. Master'dan gelen emirleri uygular (Pod'u başlatır/durdurur).
    
2. **Kube-Proxy:** Ağ trafiğini yöneten trafik polisidir.
    

---

### 3. Temel Yapı Taşı: Pod (Konteyner Değil!)

Kubernetes dünyasında en küçük birim Konteyner değildir, **Pod**'dur.

- **Nedir?** Bir veya daha fazla konteynerin paketlendiği, aynı IP adresini ve disk alanını paylaştığı kapsüldür.
    
- **Neden?** Bazen ana uygulamanın yanına yardımcı bir uygulama (Sidecar - örn: Log toplayıcı) koymak istersin. Bunlar etle tırnak gibi beraber gezmelidir. Pod bunu sağlar.
    
- **Mühendislik Kuralı:** Pod'lar ölümlüdür (Ephemeral). Bir Pod ölürse tamir edilmez, **yenisi yaratılır.**
    

---

### 4. Yönetim Objeleri: Deployment & ReplicaSet

Pod'ları asla tek tek elle yönetmeyiz. Onları yönetmesi için üst düzey objeler kullanırız.

- **ReplicaSet:** "Şu Pod'dan her zaman X tane olsun" garantisini verir. Biri silinirse hemen yenisini açar.
    
- **Deployment:** ReplicaSet'in bir üst katmanıdır. **Versiyon güncellemelerini** yönetir.
    
    - _Rolling Update:_ V1 versiyonundan V2'ye geçerken, sistemi kapatmadan Pod'ları teker teker değiştirir (**Zero Downtime**).
        

---

### 5. Networking: Service ve Ingress

Kubernetes'te ağ yönetimi en karmaşık kısımdır.

#### A. Service (Sabit Adres)

Pod'lar ölüp dirildikçe IP adresleri değişir. Frontend, Backend'i nasıl bulacak?

- **ClusterIP:** Cluster içinde geçerli sanal, sabit bir IP verir. Pod'lar değişse de bu IP değişmez.
    
- **NodePort:** Sunucunun fiziksel bir portunu dışarı açar (Örn: 30001).
    
- **LoadBalancer:** Cloud provider'ın (Azure/AWS) gerçek Load Balancer'ını tetikler.
    

#### B. Ingress (Kapı Bekçisi)

Tek tek port açmak yerine, tüm trafiği tek noktadan alıp Domain bazlı dağıtır. (L7 Load Balancer).

- `api.site.com` -> Backend Service
    
- `site.com` -> Frontend Service
    

---

### 6. Storage: Kalıcılık Problemi (PV & PVC)

Pod silindiğinde içindeki veri de silinir. Veritabanı gibi kalıcı veri gereken yerlerde ne yapacağız?

- **Persistent Volume (PV):** Fiziksel depolama alanıdır (Disk). Yönetici (Admin) tarafından tanımlanır.
    
- **Persistent Volume Claim (PVC):** Yazılımcının "Bana 10 GB yer lazım" diye oluşturduğu talep fişidir.
    
- **Süreç:** K8s, PVC talebini görür, uygun bir PV bulur ve bunları birbirine bağlar (Bind). Pod kapansa bile PV (Disk) ve veri orada kalır.
    

---

### 7. ConfigMap ve Secrets (Konfigürasyon Yönetimi)

Kodun içine şifre veya DB adresi gömmek yasaktır.

- **ConfigMap:** Şifreli olmayan ayarlar (Log seviyesi, API URL'i). Çevresel değişken (Environment Variable) veya dosya olarak Pod'a verilir.
    
- **Secrets:** Şifreli (Base64) hassas veriler (DB şifresi, API Key).
    

---

### 8. Gelişmiş Özellikler: HPA ve Namespaces

1. **Horizontal Pod Autoscaler (HPA):**
    
    - CPU veya RAM kullanımına bakar.
        
    - Yük artarsa Pod sayısını otomatik artırır (Scale Out).
        
    - Yük azalınca Pod'ları siler (Scale In).
        
2. **Namespaces (Sanal Kümeler):**
    
    - Aynı fiziksel Cluster'ı mantıksal parçalara böler.
        
    - `Dev`, `Test`, `Prod` ortamlarını aynı donanım üzerinde ama birbirini görmeyecek şekilde izole çalıştırabilirsin.