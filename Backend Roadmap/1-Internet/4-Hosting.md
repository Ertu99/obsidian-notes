**Hosting (Barındırma)**

Hosting, yazdığın kodun, veritabanının ve dosyaların (HTML, CSS, görseller) internete 7/24 bağlı bir bilgisayarda (sunucuda) saklanması ve yayınlanması hizmetidir.

Neden Kullanılır?

Kendi bilgisayarını sürekli açık tutamazsın, elektrik kesilebilir, internetin yavaşlayabilir veya IP adresin değişebilir. Hosting firmaları, yüksek hızlı internete sahip, iklimlendirmeli, yedekli elektrik sistemleri olan veri merkezlerinde (Data Center) bu hizmeti profesyonel olarak sağlar. Özetle; Domain adresin, Hosting ise o adresteki evindir.

---

### Deep Dive: Hosting Türleri ve Teknik Detaylar

Bir Backend Developer olarak "Kod bende çalışıyor" demek yetmez. O kodun canlı ortamda (Production) hangi donanım ve mimari üzerinde çalışacağını bilmek zorundasın. Mülakatlarda bu ayrımları bilmek, sistem tasarımına (System Design) hakimiyetini gösterir.

#### 1. Hosting Türleri (Eskiden Yeniye)

- **Shared Hosting (Paylaşımlı):**
    
    - **Mantık:** Bir sunucunun içinde yüzlerce site barınır. Kaynaklar (RAM, CPU) ortaktır.
        
    - **Analogy:** Öğrenci yurdunda kalmak gibidir. Biri çok gürültü yaparsa (fazla kaynak tüketirse) sen de rahatsız olursun.
        
    - **Backend Açısından:** .NET Core projeleri için pek önerilmez. Genelde PHP/WordPress siteler içindir. Özelleştirme yetkisi (yetkili kullanıcı/root) yoktur.
        
- **VPS (Virtual Private Server - Sanal Sunucu):**
    
    - **Mantık:** Fiziksel bir sunucu, yazılımsal olarak sanal parçalara bölünür. Sana ayrılan RAM ve CPU garantidir.
        
    - **Analogy:** Apartman dairesi gibidir. Bina ortaktır ama daire senindir, duvarları kalındır.
        
    - **Önem:** Junior seviyesinde projelerini yayınlamak için en iyi başlangıç noktasıdır (DigitalOcean, Linode vb.). İşletim sistemini (Linux/Ubuntu veya Windows Server) sen yönetirsin.
        
- **Dedicated Server (Fiziksel Sunucu):**
    
    - **Mantık:** Tüm fiziksel makineyi kiralarsın.
        
    - **Analogy:** Müstakil villa.
        
    - **Kullanım:** Çok yüksek trafikli siteler (Örn: E-ticaret devleri) veya bankalar gibi verinin fiziksel olarak nerede olduğunu bilmek isteyen kurumlar kullanır.
        
- **Cloud Hosting (Bulut - AWS, Azure, Google Cloud):** _Mülakatın Yıldızı_
    
    - **Mantık:** Tek bir sunucu yoktur, birbirine bağlı binlerce sunucu havuzu vardır.
        
    - **Özellik:** **Scalability (Ölçeklenebilirlik).** Trafik artarsa otomatik olarak kaynak eklenir (Auto-scaling), trafik azalınca kaynaklar düşer. Kullandığın kadar ödersin.
        
    - **NET Core İçin:** Genelde **Azure App Service** kullanılır. Sunucu yönetimiyle (OS güncellemeleri vb.) uğraşmazsın (PaaS - Platform as a Service).
        

#### 2. İşletim Sistemleri (OS)

- **Linux Hosting:** .NET Core öncesi (.NET Framework) sadece Windows'ta çalışırdı. Ama .NET Core ve sonrası (NET 5, 6, 7, 8) artık **Cross-Platform**'dur. Yani Linux sunucularda (Ubuntu, Debian) çalışabilir. Linux sunucular daha ucuz ve performanslı olduğu için sektör standardı haline gelmektedir.
    
- **Windows Hosting:** Eski .NET projeleri veya IIS (Internet Information Services) zorunluluğu olan durumlar için kullanılır.
    

#### 3. Web Server Yazılımları (İstekleri Karşılayan Kapıcılar)

Senin yazdığın C# kodu (Backend) doğrudan internete açılmaz. Önünde bir koruyucu/yönlendirici yazılım olması gerekir.

- **Kestrel:** .NET Core uygulamasının içinde gömülü gelen (internal) çok hızlı bir web sunucusudur. Ancak güvenlik özellikleri sınırlıdır, internete doğrudan açılması önerilmez.
    
- **Reverse Proxy (Ters Vekil Sunucu):** Kestrel'in önüne konulur. Dış dünyadan gelen isteği karşılar, güvenliğini kontrol eder ve Kestrel'e iletir.
    
    - Linux'ta: **Nginx** veya **Apache**.
        
    - Windows'ta: **IIS**.
        

---

### Backend Developer İçin Neden Önemli?

1. **Deployment (Dağıtım) Süreci:** Kodu yazıp bitirdiğinde onu sunucuya nasıl atacaksın? FTP ile dosya atmak eskide kaldı. Artık CI/CD (GitHub Actions, Azure DevOps) ile kodun otomatik olarak Cloud sunuculara (Hosting'e) gönderiliyor.
    
2. **Maliyet Yönetimi:** Yazdığın kod çok fazla RAM tüketiyorsa (Memory Leak), Cloud faturası şişer. Hosting maliyeti doğrudan senin kod kalitenle ilgilidir.
    
3. **Ortam Farklılıkları:** "Localhost'ta çalışıyordu ama sunucuda patladı" dememek için, uygulamanın çalışacağı Hosting ortamını (işletim sistemi, versiyonlar) bilmen gerekir.
    

