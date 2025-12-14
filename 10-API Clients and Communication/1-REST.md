İstemci (Client) tarafında `HttpClient` ile nasıl istek atacağımızı öğrendik. Şimdi masanın diğer tarafına geçiyoruz: **Sunucu (Server) tarafında, sağlam, ölçeklenebilir ve standartlara uygun bir REST API nasıl inşa edilir?**

Attığın metinde "REST bir mimari stildir" denmiş. Bu çok doğru. REST bir kural seti değildir (SOAP gibi), bir **felsefedir.** Roy Fielding'in doktora tezinde tanımladığı bu felsefe, internetin (Web) çalışma mantığıyla (HTTP) birebir uyumlu olmayı hedefler.

Bu konuyu basit bir "Controller yazma" işinden çıkarıp, **Richardson Olgunluk Modeli**, **Idempotency (Etkisizlik)** ve **Content Negotiation (İçerik Pazarlığı)** gibi kıdemli mühendislik kavramlarıyla inceleyeceğiz.

---

### 1. Felsefe: Resource-Oriented Architecture (ROA)

Eskiden (SOAP/RPC zamanlarında) API'ler "Eylem" (Verb) odaklıydı.

- `getUser()`
    
- `deleteProduct()`
    
- `calculatePrice()`
    

REST ise **"Kaynak" (Resource/Noun)** odaklıdır.

- `/users`
    
- `/products/1`
    
- `/prices`
    

**Mühendislik Kuralı:** URL'lerde asla fiil (eylem) kullanma. Eylemi **HTTP Metodu** (GET, POST, DELETE) belirler.1 URL sadece "Neyin üzerinde çalışıyoruz?" sorusunun cevabıdır (Adres).

---

### 2. Richardson Maturity Model (API'n Ne Kadar Olgun?)

Leonard Richardson, REST API'leri 4 seviyeye ayırmıştır. Bir mimar olarak hedefimiz **Level 2** veya **Level 3** olmalıdır.

- **Level 0 (The Swamp of POX):** HTTP'yi sadece tünel olarak kullanır. Tek bir URL vardır (`/api/service`) ve genelde sadece `POST` ile XML gönderilir. (Eski SOAP mantığı).
    
- **Level 1 (Resources):** URL'ler kaynaklara bölünmüştür (`/api/users`, `/api/products`). Ama hala her şey için POST kullanılır.
    
- **Level 2 (HTTP Verbs - Hedefimiz):** HTTP fiilleri (GET, POST, PUT, DELETE) doğru anlamlarıyla kullanılır. Durum kodları (200, 404, 201) doğru döner. Sektör standardı budur.
    
- **Level 3 (HATEOAS - Hypermedia):** API cevabının içinde, o veriyle yapılabilecek diğer işlemlerin linkleri de döner.
    
    - _Örnek:_ "Kullanıcıyı çektin, istersen şu linkle silebilirsin, şu linkle güncelleyebilirsin" diye cevabın içine `_links` dizisi eklenir.
        

---

### 3. Kritik Kavram: Idempotency (Etkisizlik)

Dağıtık sistemlerde ağ hatası (Network Failure) kaçınılmazdır. Bir istek attığında cevap gelmezse ne yaparsın? Tekrar atarsın (Retry).

İşte burada **Idempotency** devreye girer: **"Bir işlemi 1 kere yapmakla 100 kere yapmak arasında fark yoksa, o işlem Idempotent'tir."**

- **GET (Idempotent):** 100 kere de okusan veri değişmez. Güvenlidir.
    
- **PUT (Idempotent):** "ID:5'in adını 'Ahmet' yap" komutunu 100 kere de göndersen, sonuçta adı 'Ahmet' olur. Yan etki yoktur. Retry yapılabilir.
    
- **DELETE (Idempotent):** "ID:5'i sil" komutunu 100 kere de göndersen, sonuç (o kaydın yok olması) değişmez. İlkinde silinir, diğerlerinde 404 döner ama sistem bozulmaz.
    
- **POST (NOT Idempotent - Tehlike!):** "Sipariş Oluştur" komutunu cevap alamadığın için 2 kere gönderirsen, **2 farklı sipariş oluşur.** Müşteriden 2 kere para çekersin.
    
    - _Çözüm:_ POST isteklerinde `Idempotency-Key` header'ı kullanmak (Mühendislik çözümü).
        

---

### 4. Content Negotiation (İçerik Pazarlığı)

REST sadece JSON demek değildir. İstemci (Client) sunucudan veriyi **XML**, **CSV** hatta **Protobuf** olarak isteyebilir.

ASP.NET Core'da bu mekanizma `Accept` header'ı ile yönetilir.

- **İstemci:** `Accept: application/xml` gönderirse, API cevabı XML olarak döner.
    
- **İstemci:** `Accept: application/json` gönderirse, JSON döner.
    

Mühendislik Detayı (Protobuf):

Eğer iki mikroservis birbiriyle konuşuyorsa, JSON yavaştır (String parsing maliyeti). Accept: application/x-protobuf diyerek veriyi Binary (ikili) formatta transfer edip CPU ve Network maliyetini %90 düşürebilirsin. ASP.NET Core Formatters ile bunu destekler.

---

### 5. HTTP Status Codes (Semantik Dil)

Controller'dan sürekli `200 OK` dönmek (hata olsa bile gövdede hata mesajı dönmek) büyük bir anti-pattern'dir. HTTP kodları evrensel bir dildir.

- **2xx (Başarılı):**
    
    - `200 OK`: Standart başarı.2
        
    - `201 Created`: POST işlemi başarılı, kaynak yaratıldı. (Response Header'da `Location: /api/users/5` dönmelidir).
        
    - `202 Accepted`: İstek kuyruğa alındı, arka planda işlenecek. (Async işlemler için).
        
    - `204 No Content`: İşlem başarılı (örn: DELETE veya UPDATE) ama geriye dönecek bir veri yok.
        
- **4xx (Sen Hatalısın):**
    
    - `400 Bad Request`: Validasyon hatası.
        
    - `401 Unauthorized`: Kimlik yok (Login olmamış).
        
    - `403 Forbidden`: Kimlik var ama yetki yok.
        
    - `404 Not Found`: Kaynak yok.
        
    - `415 Unsupported Media Type`: JSON istedim, sen XML gönderdin.
        
- **5xx (Ben Hatalıyım):**
    
    - `500 Internal Server Error`: Kod patladı.
        

---

### 6. API Versioning (Versiyonlama Stratejisi)

Yazdığın API'yi mobil uygulama kullanıyor. Sen API'de bir alanı değiştirdin. Mobil uygulama güncellenmediği için patladı (Breaking Change).

Bunu önlemek için versiyonlama şarttır.

1. **URL Path (En Yaygın):** `/api/v1/products`, `/api/v2/products`.
    
    - _Avantaj:_ Çok net ve görünür.
        
2. **Query String:** `/api/products?api-version=1.0`.
    
3. **Header Versioning:** `X-Version: 1.0`. URL temiz kalır ama tarayıcıdan denemek zordur.
    

---


### 1. Sözleşme: OpenAPI (Swagger) ve "Contract First"

Bir API yazıp "Hadi kullanın" diyemezsin. Mobilciye, Frontendciye bunu nasıl kullanacağını anlatman gerekir. Word dökümanı yazmak ameleliktir ve kod değişince güncelliğini yitirir.

**OpenAPI (eski adıyla Swagger):** REST API'lerin evrensel dilidir.

- **Ne Yapar?** Kodundaki Controller'lara, modellere ve yorum satırlarına bakar; otomatik olarak **canlı bir dokümantasyon** (JSON/YAML) ve test arayüzü üretir.
    
- **Mühendislik Gücü (Code Generation):** Frontend ekibi, senin Swagger JSON dosyanı alıp `openapi-generator` aracıyla **kendi istemci kodlarını (Axios/Fetch)** tek tuşla üretebilir. Sen `User` modeline bir alan eklersin, onlar tek komutla günceller. İletişim kopukluğu biter.
    

### 2. Büyük Veri Yönetimi: Pagination, Filtering, Sorting

`/api/products` dediğinde veritabanındaki 1 milyon ürünü dönmek intihardır. REST API'de standartlaşmış kalıplar vardır:

1. **Pagination (Sayfalama):**
    
    - _Standart:_ `GET /products?page=2&pageSize=20`
        
    - _Mühendislik Detayı:_ Veritabanına `SKIP(20).TAKE(20)` olarak gider. Ayrıca Response Header'da `X-Total-Count: 1000000` dönmelisin ki istemci kaç sayfa olacağını bilsin.
        
2. **Filtering (Filtreleme):**
    
    - _Standart:_ `GET /products?category=elektronik&price_min=100`
        
    - _Mühendislik Detayı:_ Bu parametreleri dinamik olarak `IQueryable` üzerine `Where` koşulu olarak ekleyen bir altyapı kurmalısın.
        
3. **Sorting (Sıralama):**
    
    - _Standart:_ `GET /products?sort=-price,name` (Fiyat azalan, İsim artan).
        

### 3. HATEOAS (Uygulama Durum Motoru)

Richardson Modelinin 3. seviyesi demiştik. Bunu pratikte nasıl yaparsın?

Normalde JSON şöyledir:

JSON

```json
{
  "id": 1,
  "name": "Ahmet",
  "balance": 50
}
```

HATEOAS ile API, istemciye "Şimdi ne yapabilirsin?" sorusunun cevabını verir:

JSON

```json
{
  "id": 1,
  "name": "Ahmet",
  "balance": 50,
  "_links": [
    { "rel": "self", "method": "GET", "href": "/api/users/1" },
    { "rel": "delete", "method": "DELETE", "href": "/api/users/1" },
    { "rel": "top-up", "method": "POST", "href": "/api/users/1/deposit" } // Sadece yetkisi varsa görünür!
  ]
}
```

**Mühendislik Faydası:** Frontend'deki "Sil butonu aktif mi pasif mi?" mantığını Backend yönetir. Link varsa buton görünür, yoksa gizlenir.

### 4. Rate Limiting (Hız Sınırlama) - API'yi Korumak

REST API'ler halka açıktır. Kötü niyetli biri (veya bozuk bir kod) saniyede 10.000 istek atarsa sunucun çöker (DDoS).

**Çözüm:** `Asp.NetCore.RateLimiting` middleware'i.

- **Kural:** "Bir IP adresinden dakikada en fazla 60 istek gelebilir."
    
- **Cevap:** Limit aşılırsa API `429 Too Many Requests` döner ve `Retry-After: 60` header'ı ekler.
    

### 5. CORS (Cross-Origin Resource Sharing)

Browser güvenliğinin baş belasıdır ama bilmek zorundasın.

Tarayıcılar, siteA.com'daki bir JavaScript'in api.siteB.com'a istek atmasını varsayılan olarak engeller.

- **Mühendislik Ayarı:** Backend tarafında **CORS Politikası** tanımlamalısın.
    
- _"Sadece `my-app.com` ve `admin.my-app.com` domainlerinden gelen isteklere, sadece `GET` ve `POST` metodları için izin ver."_
    
- `*` (Yıldız) verip herkese açmak güvenlik açığıdır.
    

---

**🧒 6 Yaşındaki Çocuğa (Otomat Makinesi Analojisi):** "REST API'yi, AVM'deki o büyük yiyecek otomatlarına benzetebiliriz. Bu makinenin kuralları bellidir ve herkes aynı şekilde kullanır (**Standart Arayüz**):

1. Her yiyeceğin bir numarası vardır (A1, B2). Bu numaralar **URL**'dir (`/products/1`).
    
2. Camdan içeri bakmak (**GET**) bedavadır ve güvenlidir. 100 kere de baksan çikolata eksilmez (**Idempotency**).
    
3. Makineye para atıp tuşa basmak (**POST**) bir şeyleri değiştirir. Eğer makine parayı yutar ve ürün vermezse, tekrar para atarsan ikinci kez paran gider (POST güvenli değildir!).
    
4. Makinenin sana verdiği cevaplar da standarttır (**Status Codes**):
    
    - Ürün düştü: **200 OK**.
        
    - O rafta ürün yok: **404 Not Found**.
        
    - Makineye sahte para veya gazoz kapağı attın: **400 Bad Request**.
        
    - Makinenin motoru yandı, duman çıkıyor: **500 Internal Server Error**. Sen makineyle konuşmazsın, sadece düğmelere basarsın, o da sana sonucu verir."
        

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "REST, dağıtık sistemlerde kaynak (Resource) odaklı iletişim kurmanın endüstri standardıdır. Ancak 'RESTful' diyebilmek için sadece JSON dönmek yetmez; **Richardson Olgunluk Modeli**'nde en az Seviye 2'yi hedefleriz. Modern bir API mimarisinde şu standartlar esastır:

- **Semantik Doğruluk:** HTTP fiilleri (GET, POST, PUT, DELETE) ve Durum Kodları (200, 404, 500) evrensel anlamlarına uygun kullanılmalıdır. Bu, istemci tarafındaki hata yönetimini standartlaştırır.
    
- **Idempotency:** Ağ hatalarına karşı dirençli (Resilient) bir sistem için; GET, PUT ve DELETE işlemlerinin tekrarlanabilir olması, yan etki yaratmaması gerekir. POST işlemleri içinse 'Idempotency Key' mekanizmaları kurgulanır.
    
- **Documentation & Contracts:** API, **OpenAPI (Swagger)** standartlarıyla belgelenmeli ve 'Contract-First' yaklaşımıyla Frontend/Mobil ekiplerinin entegrasyonu otomatize edilmelidir.
    
- **Performance & Security:** Büyük veri setleri için **Pagination** ve **Filtering** stratejileri uygulanmalı; sistemi korumak için **Rate Limiting** middleware'leri devreye alınmalıdır."