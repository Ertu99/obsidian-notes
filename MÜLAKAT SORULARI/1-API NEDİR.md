
**API (Application Programming Interface)**,

> “Bir uygulamanın başka bir uygulama ile konuşmasını sağlayan köprüdür.”

Yani **veri alışverişi** veya **işlem yapma** amacıyla iki sistem arasında aracı görevi görür.

---

### 💡 Günlük hayattan örnek:

Bir **restorana** gittin, garsona “Bir pizza istiyorum.” dedin.

Garson (API) senin isteğini mutfağa iletir ve sonucu (pizzayı) sana getirir.

- Sen = **Frontend (kullanıcı arayüzü)**
- Mutfak = **Backend (iş mantığı ve veri tabanı)**
- Garson = **API**

---

### 💻 Teknik olarak:

Bir **API**, genellikle başka uygulamaların senin sisteminle etkileşime geçebilmesi için oluşturulan bir **arayüzdür.**

Örneğin:

```
GET <https://api.weather.com/current?city=Istanbul>

```

Bu bir hava durumu API’sidir.

Frontend bu endpoint’e istek atar, backend ona şu cevabı döner 👇

```json
{
  "city": "Istanbul",
  "temperature": 18,
  "condition": "Cloudy"
}

```

---

## 🔹 API Türleri

| Tür               | Açıklama                                                                                     |
| ----------------- | -------------------------------------------------------------------------------------------- |
| **REST API**      | En yaygın tür. HTTP protokolünü kullanır (GET, POST, PUT, DELETE). JSON formatlı veri döner. |
| **SOAP API**      | XML tabanlı, daha eski ama güvenlikli (bankacılık gibi sistemlerde hâlâ kullanılır).         |
| **GraphQL API**   | Daha esnek, istemci sadece ihtiyacı olan veriyi ister.                                       |
| **WebSocket API** | Gerçek zamanlı (ör. chat uygulamaları, borsa verileri).                                      |

---

## 🔹 REST API’de Kullanılan HTTP Metotları

|Metot|Amaç|Örnek|
|---|---|---|
|**GET**|Veri çekmek|`/api/hotels`|
|**POST**|Yeni veri eklemek|`/api/hotels`|
|**PUT**|Mevcut veriyi güncellemek|`/api/hotels/5`|
|**DELETE**|Veri silmek|`/api/hotels/5`|

---

## 💬 Kısa Özet (mülakat cevabı gibi):

> “API, iki uygulamanın birbiriyle veri alışverişi yapmasını sağlayan arayüzdür.
> 
> Günümüzde en yaygın türü REST API’dir, HTTP üzerinden çalışır ve genellikle JSON formatında veri gönderip alır.”


## 🧠 **Swagger Nedir?**

> Swagger, bir API’nin dokümantasyonunu (yani hangi endpoint’ler, hangi parametrelerle, hangi sonuçları döndürüyor) otomatik olarak oluşturan bir araçtır.

[ASP.NET](http://ASP.NET) Core’da **Swashbuckle** kütüphanesi üzerinden kullanılır.

Proje derlendiğinde Swagger otomatik olarak bir **arayüz (UI)** oluşturur ve oradan API’yi test edebilirsin.

---

## 🎯 **Swagger Ne İşe Yarar?**

| Amaç                                | Açıklama                                                                                            |
| ----------------------------------- | --------------------------------------------------------------------------------------------------- |
| 📘 **API dokümantasyonu**           | Tüm endpoint’leri, parametreleri ve dönen cevapları otomatik listeler.                              |
| 🧪 **Test kolaylığı**               | Tarayıcı üzerinden (Postman kullanmadan) doğrudan API çağrısı yapılabilir.                          |
| 🤝 **Frontend - Backend iletişimi** | Frontend geliştiriciler, API’yi rahatça inceleyip doğru şekilde entegre eder.                       |
| 🧩 **Standartlaştırma**             | API’yi “OpenAPI Specification (OAS)” formatında tanımlar.                                           |
| 🧰 **Otomasyon ve entegrasyon**     | Swagger dokümanından otomatik olarak **client kodu** (örneğin TypeScript, C#, Python) üretilebilir. |