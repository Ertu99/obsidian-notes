**Domain Name (Alan Adı)**

İnternet üzerindeki bir sunucunun veya bilgisayarın IP adresine (Örn: `142.250.185.78`) karşılık gelen, insanların kolayca okuyup ezberleyebileceği harfsel adreslerdir (Örn: `google.com`).

Neden Kullanılır?

İnsanlar karmaşık sayı dizilerini (IP adresleri) akıllarında tutamazlar. Domain isimleri, bu teknik adresleri "markalaştırır" ve erişilebilir kılar. Ayrıca, sunucunun IP adresi değişse bile domain adı aynı kalır, böylece kullanıcılar bu teknik değişiklikten etkilenmez.

---

### Deep Dive: Domain Dünyasının Teknik Anatomisi

Bir Backend Developer olarak, sadece kodu yazıp bırakmazsın; o kodun hangi domain altında, hangi subdomain ile çalışacağını da kurgulaman gerekir. Konuyu **Yapı** ve **Konfigürasyon** olarak ikiye ayıracağız.

#### 1. Domain Adının Yapısı (Hiyerarşi)

Bir domain adını sağdan sola doğru okumak gerekir. Hiyerarşi en genelden özele doğru gider.

Örnek Adres: blog.roadmap.sh

1. **Root (Kök - Gizli Nokta):** Aslında her domainin en sonunda gizli bir nokta vardır (`roadmap.sh.`). Bu, internetin kökünü temsil eder.
    
2. **TLD (Top-Level Domain - Üst Düzey Alan Adı):** Adresin en sağındaki kısımdır.
    
    - `.com`, `.net`, `.org` (Genel amaçlı - gTLD)
        
    - `.tr`, `.de`, `.uk` (Ülke kodlu - ccTLD)
        
    - Backend geliştirici olarak bilmen gereken: `.local` veya `.dev` gibi TLD'ler geliştirme ortamlarında sıkça kullanılır.
        
3. **SLD (Second-Level Domain - İkinci Düzey Alan Adı):** TLD'nin hemen solundaki kısımdır.
    
    - `roadmap` kısmı. Bu senin markandır, satın aldığın (kiraladığın) asıl kısımdır.
        
4. **Subdomain (Alt Alan Adı):** SLD'nin solundaki kısımdır.
    
    - `blog`, `api`, `admin` vb.
        
    - **Mülakat İpucu:** Backend mimarisinde Subdomainler trafiği ayırmak için kullanılır. `api.site.com` istekleri Backend sunucusuna, `www.site.com` istekleri Frontend sunucusuna yönlendirilebilir.
        

#### 2. Domain Nasıl Satın Alınır? (Registrar & Registry)

Sen `isimtescil`, `GoDaddy` veya `AWS Route53`'den domain aldığında aslında onu satın almazsın, **kiralarsın**.

- **Registry:** TLD'lerin (örneğin .com'un) veri tabanını tutan ana otoritedir. (Örn: Verisign).
    
- **Registrar:** Senin domaini kaydettirdiğin aracı firmadır (GoDaddy vb.). Sen Registrar'a para verirsin, o da Registry'ye "Bu isim artık dolu" der.
    
- **ICANN:** Tüm bu sistemin başındaki, kuralları koyan küresel organizasyondur.
    

---

#### 3. DNS Kayıtları (DNS Records) - _En Kritik Teknik Bölüm_

Backend Developer olarak bir domaini sunucuna bağlarken bilmen gereken en önemli şey **DNS Kayıt Tipleridir**. Mülakatlarda "A kaydı ile CNAME arasındaki fark nedir?" sorusu klasiktir.

Bu ayarlar Domain Kontrol Panelinden yapılır.

- **A Record (Address Record):**
    
    - **Nedir:** Domain ismini doğrudan bir **IPv4** adresine bağlar.
        
    - **Örnek:** `roadmap.sh` -> `104.21.23.45`
        
    - **Kullanım:** Ana domaini sunucuna bağlamak için kullanılır.
        
- **AAAA Record:**
    
    - **Nedir:** A kaydının aynısıdır ama **IPv6** adresleri içindir. (İnternet IPv4'ten IPv6'ya geçtikçe önemi artıyor).
        
- **CNAME Record (Canonical Name):**
    
    - **Nedir:** Bir domain ismini **başka bir domain ismine** yönlendirir (IP'ye değil!). Takma isimdir.
        
    - **Örnek:** `www.roadmap.sh` -> `roadmap.sh`
        
    - **Mülakat Cevabı:** "Eğer sunucumun IP adresi değişirse her yerde güncelleme yapmak yerine, CNAME kullanarak tek bir ana domaini işaret ederim. IP değişse bile CNAME'ler etkilenmez."
        
    - **Kısıtlama:** Ana domaine (Root domain / Naked domain -> `roadmap.sh`) CNAME verilemez, sadece Subdomainlere (`www`, `api`) verilebilir.
        
- **MX Record (Mail Exchange):**
    
    - **Nedir:** O domainin e-posta trafiğini (örn: `info@roadmap.sh`) hangi sunucunun yöneteceğini belirler.
        
    - **Önem:** Eğer uygulaman mail gönderiyorsa ve mailler spama düşüyorsa MX ve SPF kayıtlarına bakman gerekebilir.
        
- **TXT Record:**
    
    - **Nedir:** Metin tabanlı notlardır.
        
    - **Kullanım:** Genellikle doğrulama için kullanılır. Google Search Console veya SSL sertifikası alırken "Bu domain senin mi?" diye sorarlar ve senden DNS ayarlarına özel bir kod eklemeni isterler. O kod TXT kaydıdır.
        

---

### Backend Developer İçin Neden Önemli?

1. Multi-Tenancy (Çoklu Kiracı) Uygulamalar:
    
    Yazdığın SaaS projesinde her müşteriye özel subdomain verebilirsin (musteri1.uygulamam.com). .NET Core tarafında gelen isteğin Host başlığına bakarak hangi müşterinin geldiğini anlaman ve ona göre veritabanına bağlanman gerekir.
    
2. Reverse Proxy ve Hosting:
    
    Uygulamanı Nginx veya IIS arkasında yayınlarken, sunucuya "Eğer istek api.sitem.com adresinden geliyorsa 5000 portuna yönlendir" ayarını yaparsın. Bu domain adıyla yapılır.
    
3. CORS Hataları:
    
    Önceki derste konuştuğumuz CORS hataları domain bazlıdır. api.com domaini app.com domainine izin vermelidir.
    

