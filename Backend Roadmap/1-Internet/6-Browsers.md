Web tarayıcıları (Browser), kullanıcıların internet üzerindeki kaynaklara (web siteleri, resimler, videolar) erişmesini, bunları görüntülemesini ve etkileşime girmesini sağlayan yazılımlardır.

Neden Kullanılır?

Sunucular (Backend) bize 0 ve 1'lerden oluşan ham veriler, HTML kodları veya JSON dosyaları gönderir. İnsanlar bu kodları okuyup anlayamaz. Tarayıcılar, bu kodları alır, işler (render eder) ve bizim anlayabileceğimiz görsel bir arayüze (renkli butonlar, metinler, resimler) dönüştürür. Yani tarayıcı, sunucu ile insan arasındaki tercümandır.

---

### Deep Dive: Tarayıcılar Nasıl Çalışır?

Bir Backend Developer olarak genellikle "Ben API'yi yazdım, gerisi Frontend'in işi" denir ama mülakatlarda tarayıcının çalışma mantığını (özellikle motorlarını) bilmek seni rakiplerinden ayırır.

#### 1. Tarayıcının Kalbi: Motorlar (Engines)

Tarayıcılar sadece bir pencere değildir, arkada çalışan çok güçlü iki motor vardır:

- **Rendering Engine (Görüntüleme Motoru):** HTML ve CSS'i alıp ekrana çizen kısımdır.
    
    - **Blink:** Google Chrome, Edge, Opera kullanır.
        
    - **WebKit:** Apple Safari kullanır.
        
    - **Gecko:** Mozilla Firefox kullanır.
        
    - _Neden önemli?_ Bazen CSS kodun Chrome'da düzgün çalışıp Safari'de bozuk görünebilir; sebebi motorların farklı yorumlamasıdır.
        
- **JavaScript Engine (JS Motoru):** Sayfadaki etkileşimi (tıklamalar, animasyonlar) sağlayan JavaScript kodunu makine diline çevirir.
    
    - **V8 Engine:** Chrome ve Edge kullanır. (Ayrıca **Node.js** de bu motoru kullanır, bu yüzden çok ünlüdür).
        
    - **SpiderMonkey:** Firefox kullanır.
        

#### 2. Critical Rendering Path (Kritik İşleme Yolu)

Tarayıcı sunucudan cevabı aldığında (HTML), ekrana görüntüyü basana kadar şu adımları izler. Buna "Render Pipeline" denir:

1. **Parsing HTML (DOM Tree):** HTML kodunu okur ve bir ağaç yapısı (Document Object Model) oluşturur. `<div>`, `<h1>` gibi etiketler nesneye dönüşür.
    
2. **Parsing CSS (CSSOM Tree):** CSS dosyalarını okur ve stiller için ayrı bir ağaç oluşturur.
    
3. **Render Tree:** DOM ve CSSOM birleştirilir. (Örneğin, CSS'te `display: none` denilen bir eleman Render Tree'ye dahil edilmez).
    
4. **Layout (Reflow):** Her elemanın ekrandaki tam koordinatları (x, y) ve boyutu hesaplanır. (Geometri dersi burada başlar).
    
5. **Paint (Boyama):** Hesaplanan kutuların içi renklerle, resimlerle doldurulur ve pikseller ekrana basılır.
    

#### 3. Depolama Alanları (Web Storage)

Tarayıcı sadece gösterim yapmaz, aynı zamanda küçük bir veritabanı gibi davranır. Backend developer olarak buraya veri yazıp okuyabilirsin:

- **Cookies:** Genellikle kimlik doğrulama (Auth) tokenlarını saklamak için kullanılır. Her HTTP isteğiyle otomatik olarak sunucuya gönderilir.
    
- **LocalStorage:** Tarayıcı kapansa bile silinmeyen verilerdir (Örn: Sepetteki ürünler). Sadece JavaScript ile erişilir, sunucuya otomatik gitmez.
    
- **SessionStorage:** Tarayıcı sekmesi kapanınca silinen geçici verilerdir.
    

---

### Backend Developer İçin Neden Önemli?

1. DevTools (Geliştirici Araçları):
    
    Tarayıcıda F12'ye bastığında açılan panel senin en iyi dostundur.
    
    - **Network Tab:** Yazdığın API'ye giden isteği, dönen cevabı, durum kodunu (200, 404, 500) ve ne kadar sürede geldiğini buradan incelersin. API testi burada başlar.
        
2. **Güvenlik (Security):**
    
    - Tarayıcılar, senin sunucunu korumak için **CORS** politikasını zorla uygular.
        
    - Kullanıcı giriş yaptığında gönderdiğin Token'ı **HttpOnly Cookie** olarak saklamak, XSS saldırılarına karşı en güvenli yoldur. Bunu tarayıcı yönetir.
        
3. Performans:
    
    Eğer Backend'den gereksiz yere 10MB veri gönderirsen, tarayıcının bunu işlemesi (Parsing) ve boyaması (Paint) uzun sürer. Site donar. Kullanıcı "Site yavaş" der, patron Backendciye bakar.
    

