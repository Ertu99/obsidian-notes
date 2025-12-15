HTTP (Hypertext Transfer Protocol), internet üzerinde verilerin (HTML dosyaları, resimler, JSON verileri vb.) bir noktadan diğerine nasıl aktarılacağını belirleyen kurallar bütünüdür.1 Kısaca, Web'in iletişim dilidir.

Neden kullanıldığına gelirsek; tarayıcıların (Chrome, Edge) ve sunucuların (Server) birbirini anlayabilmesi için ortak bir standarta ihtiyaçları vardır. HTTP, bu standardı sağlar. Temel çalışma prensibi **Request-Response (İstek-Cevap)** modeline dayanır. Sen (Client/İstemci) bir şey istersin, sunucu (Server) da sana cevap verir ya da veremiyorsa neden veremediğini söyler. Mülakatta en önemli özelliklerinden biri olarak **"Stateless" (Durumsuz)** olduğunu belirtmelisin; yani sunucu, senin bir önceki isteğini hatırlamaz, her istek birbirindan bağımsızdır.

---

### Deep Dive: HTTP'nin Anatomisi

Backend geliştirici olarak kod yazarken aslında yaptığın işin %90'ı HTTP mesajlarını yönetmektir. .NET Core'da yazdığın bir Controller aslında HTTP isteğini karşılayıp bir HTTP cevabı üretir.

#### 1. HTTP Request (İstek) Yapısı

İstemci (Client) sunucuya bir istek gönderdiğinde, bu istek üç ana bölümden oluşur:2

- **Start Line (Başlangıç Satırı):** Hangi eylemi yapmak istediğini (Method) ve nereye gitmek istediğini (URL) belirtir.3
    
    - Örnek: `GET /api/products HTTP/1.1`
        
- **Headers (Başlıklar):** İsteğin "kimliği" ve "meta verileridir".4
    
    - `Host: roadmap.sh` (Hangi siteye gidiyorum?)
        
    - `Content-Type: application/json` (Sana JSON formatında veri gönderiyorum)5
        
    - `Authorization: Bearer xyztoken` (Benim girişim var, bu da biletim)
        
- **Body (Gövde):** Sunucuya gönderilen asıl veridir. (GET isteklerinde genellikle boştur, POST veya PUT isteklerinde kayıt edilecek veri burada durur).
    

#### 2. HTTP Methods (Fiiller) - _Çok Kritik_

Backend developer olarak API yazarken "Hangi işlemi yapıyorsam ona uygun methodu seçmeliyim" kuralına uymak zorundasın. Buna **Semantik** denir.

- **GET:** Veri okumak/çekmek için kullanılır. Sunucuda bir şeyi değiştirmez. (Örn: Ürün listesini getir).
    
- **POST:** Yeni bir veri oluşturmak (Create) için kullanılır.6 (Örn: Yeni ürün ekle).
    
- **PUT:** Var olan bir veriyi **tamamen** güncellemek için kullanılır. (Örn: Ürünün adını, fiyatını, stoğunu hepsini birden güncelle).
    
- **PATCH:** Var olan bir veriyi **kısmi** güncellemek için kullanılır.7 (Örn: Sadece ürünün fiyatını değiştir).
    
- **DELETE:** Veriyi silmek için kullanılır.
    

#### 3. HTTP Response (Cevap) Yapısı

Sunucu işini bitirince sana bir paket hazırlar:

- **Status Code (Durum Kodu):** İşlemin sonucunu bildiren 3 haneli sayı.8
    
- **Headers:** Sunucunun gönderdiği meta veriler (Örn: `Server: Kestrel`, `Date: ...`).9
    
- **Body:** İstenen veri (Örn: Ürünlerin JSON listesi veya HTML sayfası).10
    

#### 4. HTTP Status Codes (Durum Kodları) - _Mülakat Sorusu_

Bu kodları ezberlemene gerek yok ama grupları bilmelisin:

- **2xx (Başarılı):** İşlem tamam.
    
    - **200 OK:** Her şey yolunda.
        
    - **201 Created:** Başarıyla oluşturuldu (Genelde POST sonrası döner).11
        
- **3xx (Yönlendirme):** Aradığın şey başka yere taşındı.
    
- **4xx (İstemci Hatası):** Sorun sende (Client).
    
    - **400 Bad Request:** Hatalı istek attın, veriler eksik veya yanlış.
        
    - **401 Unauthorized:** Giriş yapmamışsın, kim olduğunu bilmiyorum.
        
    - **403 Forbidden:** Giriş yapmışsın ama buraya girmeye yetkin yok.12
        
    - **404 Not Found:** İstediğin şey sunucuda yok.
        
- **5xx (Sunucu Hatası):** Sorun bende (Server).
    
    - **500 Internal Server Error:** Kodum patladı, sunucuda hata var.
        

#### 5. Stateless (Durumsuz) Olmak Ne Demek?

HTTP protokolü "unutkandır". Sunucuya "Selam ben Ali" dersin (İstek 1), sunucu "Selam Ali" der. Hemen ardından "Benim adım ne?" diye sorarsan (İstek 2), sunucu "Bilmiyorum" der.

Çünkü 2. istek, 1. isteği bilmez.

- **Çözüm:** Backend'de bu sorunu aşmak için **Cookie**, **Session** veya **Token (JWT)** kullanırız.13 Her istekte kimliğimizi tekrar göndeririz ki sunucu bizi hatırlasın.
    

#### 6. HTTPS (Secure)

HTTP şifrelenmemiş metindir (Plain Text).14 Eğer biri araya girerse (Man-in-the-Middle), gönderdiğin şifreyi veya kredi kartını açıkça okuyabilir.15

- **HTTPS:** HTTP'nin SSL/TLS protokolü ile şifrelenmiş halidir.16 Veriler okunamaz hale gelir. Google artık HTTPS olmayan siteleri güvensiz olarak işaretliyor.
    

---

### Backend Developer İçin Neden Önemli?

Sen bir .NET developer olarak `Controller` yazdığında aslında HTTP protokolünü yönetiyorsun.

- Eğer bir veri kaydederken `GET` metodu kullanırsan veya başarılı işlem sonucunda `200 OK` yerine `404` dönersen, tarayıcı (veya Frontend developer) uygulamanın bozuk olduğunu düşünür.
    
- REST API standartlarına uymak demek, HTTP methodlarını ve Status kodlarını doğru kullanmak demektir.
    


#### 1. HTTP Versiyonları (Mülakatta Fark Yaratır)

İnternet geliştikçe HTTP de güncellendi. Farkları bilmek, performans vizyonunu gösterir.

- **HTTP/1.1 (Eski Standart - Text Based):**
    
    - Sıralı işlem yapar. Bir resim yüklenmeden diğeri başlamaz (Head-of-Line Blocking).
        
    - Metin tabanlıdır (insan okuyabilir).
        
    - Her dosya için ayrı TCP bağlantısı açmaya meyillidir.
        
- **HTTP/2 (Modern Standart - Binary):**
    
    - **.NET Core varsayılan olarak destekler.**
        
    - **Multiplexing (Çoklama):** Tek bir TCP bağlantısı üzerinden aynı anda birden fazla dosya (resim, css, js) gönderilebilir. Trafik sıkışıklığı olmaz.
        
    - **Binary:** Veriler 1 ve 0 (binary) formatında taşınır, bu yüzden çok daha hızlıdır ve hata oranı düşüktür.
        
    - **Server Push:** İstemci istemeden sunucu gerekli dosyaları (örn: style.css) önceden gönderebilir.
        
- **HTTP/3 (Gelecek - UDP Tabanlı):**
    
    - TCP yerine QUIC (UDP) protokolünü kullanır. Henüz her yerde yok ama "biliyorum" demek çok havalı durur.
        

#### 2. CORS (Cross-Origin Resource Sharing) - _Mülakat ve Gerçek Hayatın Belası_

Junior developerların işe başladığında ilk haftasında en çok "patladığı" konu budur.

- **Sorun Nedir?** Tarayıcılar güvenlik gereği "Same-Origin Policy" (Aynı Köken Politikası) uygular. Eğer senin React uygulaman `localhost:3000`'de çalışıyor ama .NET API'n `localhost:5000`'de çalışıyorsa, tarayıcı varsayılan olarak React'ın API'ye erişmesini **engeller**. Çünkü portları farklıdır, yani "farklı köken" (Cross-Origin) sayılırlar.
    
- **Hata Mesajı:** Konsolda kırmızı renkte _"Access to fetch at ... has been blocked by CORS policy"_ yazar.
    
- **Çözüm (.NET Core'da):** Backend developer olarak sunucuya "Korkma, `localhost:3000` güvenlidir, ona cevap ver" demen gerekir. Bunu .NET Core'da `Program.cs` içinde **CORS Policy** ekleyerek yaparsın.
    
    _Mülakat Cevabı:_ "Frontend ve Backend farklı domain veya portlarda çalışıyorsa tarayıcı güvenlik engelini açmak için Backend tarafında CORS (Cross-Origin Resource Sharing) ayarlarını yapılandırır, izin verilen domainleri (Origins) belirtirim."