
### ACELOT

Şimdi mikroservislerin **"Ön Kapısı"na**, kaosun düzenle buluştuğu noktaya, **API Gateway** ve onun .NET dünyasındaki sancaktarı **Ocelot**'a geliyoruz.

Mikroservis mimarisinde arkada 50 tane servis çalışır.

- `CatalogService`: `localhost:5001`
    
- `OrderService`: `localhost:5002`
    
- `PaymentService`: `localhost:5003`
    

Mobil uygulama geliştiricisine "Al bu 50 tane IP adresini, tek tek istek at" dersen, proje batar. API Gateway, bu 50 adresi tek bir adreste (`api.mysite.com`) toplayan ve trafiği yöneten **Trafik Polisi**dir.

Bu konuyu; **Routing (Yönlendirme)**, **Aggregation (Birleştirme)**, **Service Discovery Entegrasyonu** ve **BFF (Backend for Frontend)** deseni üzerinden eksiksiz inceleyelim.

---

### 1. Felsefe: Reverse Proxy Nedir?

Ocelot temelde bir **Reverse Proxy** (Ters Vekil Sunucu) olarak çalışır.

- **Forward Proxy:** Sen (İstemci) internete çıkarken kimliğini gizlemek için kullanırsın (VPN gibi).
    
- **Reverse Proxy:** Sunucular (Servisler) dış dünyadan gizlenmek için kullanır.
    
    - İstemci isteği Ocelot'a atar.
        
    - Ocelot "Bu istek Sipariş servisine gitmeli" der ve isteği arkadaki sunucuya iletir.
        
    - Cevabı alır ve istemciye döner.
        
    - **Fayda:** İstemci, arkada kaç tane servis olduğunu, IP adreslerini veya teknolojilerini (Java mı .NET mi?) asla bilemez. Güvenlik sağlar.
        

![API Gateway architecture pattern resmi](https://encrypted-tbn2.gstatic.com/licensed-image?q=tbn:ANd9GcRqG-X3rK3J9IbwArRJtNIB3q1d_XxWzhLZaD2Cm3yrKXTC-774P6HcVImFBm_GbjTPUsxhm5z6MXrAC9bwitaC8VsEQ7gCsRgRfN6Yu9BnNEtuSLA)

Shutterstock

---

### 2. Yapı Taşı: `ocelot.json`

Ocelot kod yazarak değil, **Konfigürasyon (JSON)** ile yönetilir. İki temel kavram vardır:

1. **Upstream (Yukarı Akış):** İstemcinin gördüğü URL. (Senin API Gateway adresin).
    
2. **Downstream (Aşağı Akış):** Gerçek mikroservisin adresi.
    

JSON

```json
{
  "Routes": [
    {
      // İstemci buna istek atacak: api.com/products
      "UpstreamPathTemplate": "/products",
      "UpstreamHttpMethod": [ "Get" ],

      // Ocelot isteği buraya yönlendirecek: localhost:5001/api/v1/catalog
      "DownstreamPathTemplate": "/api/v1/catalog",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "localhost", "Port": 5001 }
      ]
    }
  ]
}
```

---

### 3. Mühendislik Harikası: Request Aggregation (İstek Birleştirme)

Mobil uygulamalar "Chatty" (Geveze) olmayı sevmez. Tek bir ekranı çizmek için 5 farklı servise istek atmak, pil ömrünü ve veri paketini yer.

**Senaryo:** "Ana Sayfa"da hem kullanıcının adını (UserService), hem de son siparişlerini (OrderService) göstereceksin.

- **Ocelot Olmadan:** Mobil uygulama 2 istek atar.
    
- **Ocelot Aggregation:**
    
    - Mobil uygulama tek istek atar: `GET /home-summary`
        
    - Ocelot arkada paralel olarak hem User hem Order servisine gider.
        
    - İki JSON cevabını birleştirir (`Merge`).
        
    - Tek bir JSON döner: `{ "user": {...}, "orders": [...] }`
        

**Not:** Bu işlem Ocelot config dosyasında tanımlanır ama çok karmaşık senaryolar için kod yazmak (Custom Aggregator) gerekebilir.

---

### 4. Service Discovery Entegrasyonu (Consul / Eureka)

Yukarıdaki JSON örneğinde `localhost:5001` yazdık. Bu bir **Monolith alışkanlığıdır.** Mikroservis dünyasında, Kubernetes sürekli yeni Pod açar, eskilerini kapatır. IP adresleri ve Portlar her saniye değişir. Elle IP yazamazsın.

**Çözüm:** Ocelot + **Consul** (veya Kubernetes DNS).

1. Mikroservisler ayağa kalkınca Consul'a "Ben CatalogService, IP'm bu" diye kayıt olur.
    
2. Ocelot konfigürasyonuna IP yazmazsın, sadece Servis Adı yazarsın: `"ServiceName": "CatalogService"`.
    
3. İstek geldiğinde Ocelot, Consul'a sorar: "CatalogService şu an nerede?"
    
4. Consul güncel adresi verir, Ocelot oraya gider.
    

---

### 5. Cross-Cutting Concerns (Yükü Atmak)

Mikroservislerin her birine ayrı ayrı "Token Doğrulama", "Rate Limiting", "CORS" kodu yazmak **DRY (Don't Repeat Yourself)** prensibine aykırıdır. Bu işleri Gateway yapar.

1. **Authentication:** Ocelot, gelen istekteki JWT Token'ı doğrular. Geçersizse `401` döner. Arkadaki servise istek bile gitmez. Servislerin yükü hafifler.
    
2. **Rate Limiting:** "Bu kullanıcı saniyede 5 istekten fazla atamaz" kuralını Ocelot'a koyarsın. Servislerin DDoS saldırısından korunur.
    
3. **Caching:** Bazı cevapları Ocelot önbelleğe alabilir. Arkadaki servise hiç gitmeden cevap döner.
    

---

### 6. Mimari Desen: BFF (Backend For Frontend)

Bazen tek bir API Gateway yetmez.

- **Mobil Uygulama:** Az veri ister (Hız önemli).
    
- **Web Dashboard:** Çok detaylı veri ister (Analiz önemli).
    

Eğer tek Gateway kullanırsan, Mobil için Web verilerini de çekmek zorunda kalırsın (Over-fetching).

**Çözüm:** İki ayrı Ocelot Gateway kurarsın.

1. **Mobile Gateway:** Sadece mobilin ihtiyacı olan verileri, küçük paketler halinde sunar.
    
2. **Web Gateway:** Zengin veriler sunar. Bu desene **BFF (Backend For Frontend)** denir ve Ocelot buna çok uygundur.
    

---

### 7. Load Balancing (Yük Dengeleme)

Eğer `CatalogService`'ten 3 tane varsa (Port 5001, 5002, 5003), Ocelot gelen istekleri bunlara dağıtabilir.

- **RoundRobin:** Sırayla.
    
- **LeastConnection:** En boş olana.



### YARP

Şimdi Ocelot'un tahtını sallayan, Microsoft'un bizzat geliştirip "Biz Azure'da bunu kullanıyoruz" dediği, modern ve ultra performanslı Reverse Proxy kütüphanesine, **YARP (Yet Another Reverse Proxy)** projesine geçiyoruz.

Ocelot, topluluk destekli (Community Driven) ve bazen güncellemelerde geri kalan bir projeydi. Microsoft, .NET Core'un performansını (Kestrel) %100 kullanabilmek için YARP'ı geliştirdi.

Bu konuyu; **Ocelot ile Farkları**, **Mimari Yapı (Routes & Clusters)**, **Dinamik Konfigürasyon** ve **Direct Forwarding** gücü üzerinden tek seferde ve eksiksiz inceleyelim.

---

### 1. Felsefe: Neden YARP? (Ocelot vs YARP)

Mülakatlarda veya mimari toplantılarda bu soru kesin gelir.

|**Özellik**|**Ocelot**|**YARP**|
|---|---|---|
|**Köken**|Topluluk (Open Source)|Microsoft (Resmi Destek)|
|**Performans**|İyi (Ama eski middleware yapısı)|**Çok Yüksek** (Kestrel üzerine optimize)|
|**Protokoller**|HTTP/1.1 (gRPC desteği sınırlı)|**HTTP/2, gRPC, WebSockets** (Kusursuz)|
|**Konfigürasyon**|JSON odaklı (`ocelot.json`)|**Kod odaklı** (C# ile tam kontrol)|
|**Kullanım**|Basit Gateway senaryoları|Karmaşık, yüksek trafikli, dinamik sistemler|

**Mühendislik Kararı:** Eğer sıfırdan proje başlıyorsan ve .NET 6/7/8 kullanıyorsan, tartışmasız **YARP** seçilmelidir.

---

### 2. Mimari: Routes ve Clusters

YARP terminolojisi Ocelot'tan biraz farklıdır. İki temel bileşen vardır:

1. **Routes (Rotalar):** İstemciden gelen isteklerin kuralları.
    
    - _"Eğer `/api/products` gelirse..."_
        
2. **Clusters (Kümeler):** Arkadaki mikroservislerin (Destination) grubu.
    
    - _"...bunu `CatalogCluster` grubuna gönder."_
        
    - `CatalogCluster` içinde 3 farklı IP (Replica) olabilir: `10.0.0.1`, `10.0.0.2`...
        

JSON

```json
// appsettings.json (YARP)
"ReverseProxy": {
  "Routes": {
    "route1": {
      "ClusterId": "catalog_cluster", // Eşleşme
      "Match": { "Path": "/products/{**catch-all}" }
    }
  },
  "Clusters": {
    "catalog_cluster": {
      "Destinations": {
        "destination1": { "Address": "https://localhost:5001" }, // Servis A
        "destination2": { "Address": "https://localhost:5002" }  // Servis B (Yedek)
      }
    }
  }
}
```

---

### 3. Mühendislik Harikası: Code-Based Configuration (Dinamik Yapı)

Ocelot'ta yeni bir servis eklemek için `ocelot.json` dosyasını değiştirip uygulamayı **restart** etmen gerekirdi (veya karmaşık db entegrasyonu yapardın).

YARP, **"IProxyConfigProvider"** arayüzü ile gelir.

- Sen bir C# sınıfı yazarsın.
    
- Bu sınıf, rotaları SQL Server'dan, Redis'ten veya Kubernetes API'den okuyabilir.
    
- Veri tabanında bir kayıt değiştiğinde, YARP **hafızadaki rotaları (In-Memory)** canlı olarak günceller. Restart gerekmez! **Zero Downtime Configuration.**
    

C#

```cs
public class MySqlConfigProvider : IProxyConfigProvider
{
    // Veritabanından rotaları oku ve YARP'a ver
    public IProxyConfig GetConfig() => _repo.GetRoutesFromDb();
}
```

---

### 4. Direct Forwarding (`IHttpForwarder`)

Bazen tam teşekküllü bir Gateway istemezsin. Kendi yazdığın bir Controller'ın içinden, gelen isteği "olduğu gibi" başka bir yere fırlatmak istersin.

YARP, çekirdek motorunu (`IHttpForwarder`) dışarı açar.

C#

```cs
// Custom Controller içinde
public async Task ForwardRequest()
{
    // Gelen isteği al, üzerinde oynama yap, Google'a fırlat
    await _forwarder.SendAsync(HttpContext, "https://google.com", _httpClient);
}
```

Bu, kendi özel Proxy sunucunu yazman için inanılmaz bir esneklik sağlar.

---

### 5. Yük Dengeleme ve Health Checks

YARP, Ocelot'tan çok daha gelişmiş algoritmalar sunar:

- **Power of Two Choices:** Rastgele 2 sunucu seçer, hangisi daha az yoğunsa ona atar (En verimli algoritmalardan biridir).
    
- **Active Health Checks:** YARP, arkadaki servislere sürekli "Yaşıyor musun?" diye ping atar. Eğer servis çökerse, trafik göndermeyi **otomatik keser.** Ocelot'ta bu özellik sonradan eklense de YARP kadar native değildir.
    

---

### 6. gRPC ve HTTP/2 Desteği

Mikroservisler arası iletişimde gRPC kullanıyorsan, Ocelot bazen baş ağrıtabilir (HTTP/1.1 dönüşümleri yüzünden).

YARP ise Kestrel üzerine kurulu olduğu için End-to-End HTTP/2 destekler.

- İstemci -> (HTTP/2) -> YARP -> (HTTP/2) -> Mikroservis.
    
- Hiçbir protokol düşürme (Downgrade) işlemi olmaz. Performans kaybı sıfırdır.





**🧒 6 Yaşındaki Çocuğa (Resepsiyonist Analojisi):** "Büyük bir otele gittiğini düşün. Otelde yüzlerce oda var (Mikroservisler). Sen içeri girdiğinde hangi odanın boş olduğunu, hangi odanın temiz olduğunu veya 505 numaralı odanın yerini bilemezsin. Girişteki **Resepsiyonist (API Gateway)** sana yardım eder. Sen ona 'Ben uyumak istiyorum' dersin. O seni 505'e gönderir. 'Ben yemek yemek istiyorum' dersin. O seni restorana gönderir. Sen odaların yerini ezberlemek zorunda kalmazsın, sadece resepsiyonistle konuşursun. Ayrıca otele kötü niyetli biri girmek isterse (**Security/Auth**), resepsiyonist onu daha kapıdayken durdurur, odalara yaklaştırmaz bile. Eskiden bu resepsiyonist her şeyi deftere yazardı (**Ocelot/JSON**), yeni resepsiyonist ise süper bir bilgisayar kullanıyor ve çok daha hızlı çalışıyor (**YARP**)."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "Mikroservis mimarilerinde, istemcilerin (Client) onlarca farklı servisle doğrudan muhatap olması; güvenlik zaafiyeti, yönetim zorluğu ve 'Chatty' (çok konuşan) ağ trafiği sorunları yaratır. Bu kaosu yönetmek için **API Gateway Pattern** uygulanması bir endüstri standardıdır. Teknoloji seçiminde yaklaşımım şöyledir:

- **Legacy/Simple Scenarios:** Basit ve JSON tabanlı konfigürasyon gerektiren projelerde **Ocelot** yeterli bir Reverse Proxy çözümüdür. Routing ve Aggregation ihtiyaçlarını karşılar.
    
- **Modern/High-Performance:** Ancak Microsoft'un resmi olarak desteklediği ve Kestrel altyapısı üzerine inşa ettiği **YARP (Yet Another Reverse Proxy)**, modern projeler için birincil tercihimdir.
    
- **Why YARP?:** YARP, HTTP/2 ve gRPC protokollerini native desteklemesi, dinamik konfigürasyon (IProxyConfigProvider) sayesinde 'Zero Downtime' güncelleme imkanı sunması ve yüksek performansı ile Ocelot'un mimari darboğazlarını aşmaktadır.
    
- **Cross-Cutting Concerns:** Gateway katmanında sadece yönlendirme değil; Authentication (Token doğrulama), Rate Limiting ve Service Discovery entegrasyonlarını da merkezi olarak çözerek, arkadaki mikroservislerin yükünü hafifletirim."