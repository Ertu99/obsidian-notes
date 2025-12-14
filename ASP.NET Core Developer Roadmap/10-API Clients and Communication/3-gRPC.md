 REST ve GraphQL ile dış dünyaya açıldık. Şimdi ise mikroservis dünyasının otobanına, **gRPC (Google Remote Procedure Call)** teknolojisine giriyoruz.

REST (JSON/HTTP1.1), insanlar okuyabilsin diye tasarlanmış metin tabanlı (text-based) bir protokoldür. Ancak "Benim iki sunucum (Mikroservisim) birbiriyle konuşurken neden insan gibi konuşsun? Makine gibi (Binary) konuşsunlar!" dersen, devreye gRPC girer.

gRPC; **Protocol Buffers (Protobuf)** ve **HTTP/2** teknolojileri üzerine kurulu, ultra düşük gecikme (Low Latency) ve yüksek performans sağlayan bir framework'tür. Özellikle mikroservisler arası iletişimde (Internal Traffic) endüstri standardıdır.

Bu konuyu sadece "hızlı bir API" olarak değil; **Binary Serialization**, **Multiplexing** ve **Streaming** yetenekleri üzerinden, tam bir mühendislik derinliğinde inceleyelim.

---

### 1. Felsefe: JSON vs Protobuf (Binary Serialization)

REST API'ler veriyi taşımak için **JSON** kullanır. JSON güzeldir ama şişmandır.

- **JSON:** `{ "ad": "Ahmet", "yas": 25 }` (Her seferinde "ad" ve "yas" stringlerini de taşır).
    
- **Protobuf:** Veriyi **Binary (0 ve 1)** formatına çevirir. Şema (Schema) önceden bellidir.
    
    - _Sunucu A:_ "1 numaralı alana Ahmet yaz."
        
    - _Sunucu B:_ "Tamam, 1 numaralı alanın 'Ad' olduğunu biliyorum."
        
    - **Sonuç:** Veri boyutu %60-%80 küçülür. Serileştirme (Serialization) hızı JSON'dan 10 kat daha hızlıdır.
        

---

### 2. Otoban: HTTP/2 ve Multiplexing

REST genelde HTTP/1.1 kullanır. gRPC ise **HTTP/2** zorunluluğu ile gelir. Bu, performansın ikinci sırrıdır.

- **HTTP/1.1 (REST):** Her istek için yeni bir TCP bağlantısı açılır (veya sıraya girer - Head of Line Blocking).
    
- **HTTP/2 (gRPC):** Tek bir TCP bağlantısı üzerinden, aynı anda yüzlerce istek/cevap paralel olarak (Multiplexing) akar.
    
    - Ayrıca Header sıkıştırması (HPACK) yaparak ağ trafiğini daha da azaltır.
        

---

### 3. Contract First Development (Sözleşme Odaklı)

gRPC'de "Code First" yoktur, "Contract First" vardır.

Kod yazmadan önce, .proto uzantılı bir dosya açar ve API'nin kurallarını yazarsın.

Protocol Buffers

```
// user.proto
syntax = "proto3";

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  int32 id = 1; // 1 numara çok önemli!
}

message UserResponse {
  string name = 1;
  int32 age = 2;
}
```

**Mühendislik Gücü (Polyglot):** Bu `.proto` dosyasını alıp, **C#** projesine koyarsan C# kodu üretir. **Go** projesine koyarsan Go kodu üretir. Dil bağımsızdır. .NET Core'da `Grpc.Tools` paketi, build anında bu dosyadan `UserBase` sınıflarını otomatik üretir.

---

### 4. İletişim Modelleri (Streaming)

REST'te sadece "İstek at -> Cevap al" vardır. gRPC 4 farklı model sunar:

1. **Unary RPC:** Klasik. 1 İstek -> 1 Cevap.
    
2. **Server Streaming:** 1 İstek -> N Cevap.
    
    - _Senaryo:_ "Bana borsa verilerini gönder." Sunucu bağlantıyı açık tutar ve sürekli fiyat akıtır.
        
3. **Client Streaming:** N İstek -> 1 Cevap.
    
    - _Senaryo:_ IoT cihazı gün boyu sıcaklık verisi gönderir, gün sonunda sunucu "Tamam, kaydettim" der.
        
4. **Bi-Directional Streaming (Çift Yönlü):** N İstek <-> N Cevap.
    
    - _Senaryo:_ Gerçek zamanlı chat uygulaması veya online oyun. İki taraf da birbirini beklemeden konuşur.
        

---

### 5. Tarayıcı Sorunu: gRPC-Web

Mülakat sorusu: _"gRPC bu kadar harikaysa neden her yerde (Browser) kullanmıyoruz?"_

Sorun: Tarayıcılar (Chrome, Firefox) henüz HTTP/2 üzerinden ham gRPC çağrıları yapmayı tam desteklemez (Binary framing kontrolü yoktur).

Çözüm: gRPC-Web.

- Araya bir Proxy (Envoy veya .NET Core Middleware) koyarsın.
    
- Browser -> (gRPC-Web / HTTP1.1) -> Proxy -> (gRPC / HTTP2) -> Backend.
    
- Ancak bu dönüşüm performansı biraz düşürür. Bu yüzden gRPC asıl gücünü **Backend-to-Backend** iletişimde gösterir.
    

---

### 6. Performans Karşılaştırması

|**Özellik**|**REST**|**gRPC**|
|---|---|---|
|**Format**|JSON / XML (Metin)|Protobuf (Binary)|
|**Protokol**|HTTP/1.1 (Genelde)|HTTP/2 (Zorunlu)|
|**Okunabilirlik**|İnsan okuyabilir|Makine okur (Binary)|
|**Hız**|Yavaş / Orta|Çok Hızlı|
|**Streaming**|Yok (veya zor)|Yerleşik (Bi-directional)|
|**Kullanım**|Public API, Browser|Microservices, IoT, Mobile|

---
