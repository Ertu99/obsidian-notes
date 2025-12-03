## 0.1 – Monolit Nedir?

**Monolit uygulama:**

Tüm sistem **tek bir uygulama** içinde:

- Tek codebase
- Tek database
- Tek deploy

Mesela:

- `EcommerceApp`
    - Controllers (OrderController, PaymentController, ProductController…)
    - Services (OrderService, PaymentService…)
    - Repositories
    - Hepsi tek proje, tek solution, tek DB: `EcommerceDb`

### Avantajları

- Başlangıçta kurması çok kolay
- Debug / geliştirme basit
- Deployment: “publish et, sunucuya at, bitti”
- Küçük ekip ve küçük proje için idealler

### Dezavantajları (asıl mikroservisi doğuran şeyler)

Proje büyüdükçe:

1. **Kod karmaşası**
    - Order ile alakasız Payment kodu aynı codebase’de
    - Bir yeri değiştirince başka yerler bozulabiliyor
2. **Tek deploy noktası**
    - Küçük bir değişiklik için bile tüm sistemi publish etmen gerekiyor
    - Hata olursa her şey down
3. **Performans / ölçekleme**
    - Sadece Payment kısmına yük biniyor ama tüm uygulamayı scale etmek zorundasın
    - Yani “Order az kullanılıyor, Payment çok kullanılıyor” diyemiyorsun → hepsi aynı kutu
4. **Tek DB – coupling**
    - Tüm domainler aynı database’i paylaşıyor
    - Her yer her tablonun CRUD’unu yapabiliyor → **bağımlılık cehennemi**
5. **Ekip büyüdükçe**
    - 20 kişi aynı codebase üzerinde çalışıyor
    - Merge conflict, koordinasyon, ownership problemleri

> Özet: Monolit, küçükken güzel, büyüyünce boğuyor.

---

## 0.2 – Mikroservis Nedir?

En sade tanım:

> Mikroservis: İşin belli bir bölümünden sorumlu, küçük, bağımsız deploy edilen, kendi verisini yöneten servisler topluluğu.

Yani:

- Order başka uygulama
- Payment başka uygulama
- Inventory başka uygulama
- Notification başka uygulama

Hepsinin:

- **Kendi kodu**
    
- **Kendi veritabanı**
    
- **Kendi deploy’u**
    
    var.
    

Bizim projede:

- `OrderService` → sadece sipariş
- `PaymentService` → sadece ödeme
- `InventoryService` → sadece stok
- `NotificationService` → sadece bildirim

Her biri **farklı .NET Web API** olacak.

---

## 0.3 – Mikroservisin Temel Prensipleri

### 1️⃣ Bounded Context (Sınır çizmek)

Her mikroservisin **net sınırı** olmalı:

- OrderService → sipariş oluşturma, sipariş durumu, geçmiş siparişler
- PaymentService → ödeme almak, iade, ödeme kaydı
- InventoryService → stok, depo
- NotificationService → mail, sms, push

**OrderService gidip Payment DB’sinden direkt veri okuyamaz.**

Her biri kendi “dünyasında”.

> Bu kavrama DDD’de “bounded context” deniyor.

✅ Biz ne yapacağız?

Order’ın DB’sinde **payment tablosu olmayacak**. Payment bilgisi için `PaymentService` ile konuşacağız (REST/event).

---

### 2️⃣ Her Servisin Kendi Veritabanı

Çok kritik kural:

> Her mikroservisin kendi veritabanı vardır.
> 
> Servisler birbirinin veritabanına direkt erişmez.

Mesela:

- OrderService → `OrderDb` (PostgreSQL)
- PaymentService → `PaymentDb`
- InventoryService → `InventoryDb`

Neden?

- Böylece şema değişikliğini bağımsız yapabilirsin
- Bir servisi başka teknolojiye taşıyabilirsin (Postgres → Mongo gibi)
- Servisler loosely coupled olur (zayıf bağ)

Biz projede **her servise ayrı Postgres DB** kuracağız.

---

### 3️⃣ Bağımsız Deploy

Mikroservislerin en büyük power’ı:

> Bir servisi güncellerken diğerlerini deploy etmene gerek yok.

Örneğin:

- Payment’te bir bug fix yaptın
- Sadece `PaymentService` imajını build + deploy edersin
- Order / Inventory / Notification aynen kalır

Monolit’te ne oluyordu?

→ ufak bir bug fix = komple uygulamayı publish et.

---

### 4️⃣ Bağımsız Ölçeklenme

Her servis **ayrı ayrı scale edilebilir**:

- Payment servisinde CPU %90 → Payment replica sayısını 1’den 5’e çıkar.
- Notification servisi çok az kullanılıyorsa 1 instance kalsın.

Konteyner tarafında:

- `order-service` → 2 container
- `payment-service` → 5 container
- `inventory-service` → 2 container
- `notification-service` → 1 container

Bu da **mikroservislerin asıl ekonomik gücü**.

---

### 5️⃣ Servisler Arası İletişim (Sync / Async)

Mikroservisler birbirine **“metot çağırır gibi”** değil, **network üzerinden** konuşur.

İki stil var:

### a) Senkron (Sync) – REST / gRPC

- `OrderService` diyor ki:
    - “Ödeme alman lazım” → `POST <http://payment/api/payments`>
- Hemen cevap bekliyor:
    - 200 OK → ödeme başarılı
    - 400 / 500 → hatalı

Bu, normal REST çağrısı.

**Sorun:**

Payment down ise Order da çaresiz kalıyor → sistem sıkı bağlı oluyor.

### b) Asenkron (Async) – Event Driven (RabbitMQ vs.)

- Order, “OrderCreated” event’i yayınlar
- Payment bu event’i **dinler** ve ödeme sürecini başlatır
- Order, Payment’ın anında cevap vermesini beklemez
- Sonuç (başarılı/başarısız) başka bir event olarak gelir

Biz projede:

- İlk olarak REST’i kısa göstereceğiz (dezavantajı anla diye)
- Sonra RabbitMQ ile **Event-Driven Architecture**’a geçeceğiz

Burası zaten sonraki teorik bölümümüz olacak.

---

## 0.4 – CAP Teoremi ve Eventual Consistency (Korkutmayacak Özet)

Microservice = dağıtık sistem.

Dağıtık sistemlerde şu konu ortaya çıkar:

> Her şey her an %100 senkron tutarlı olamaz.

### CAP Teoremi (basit, sezgisel anlatım)

3 özellik var:

- **C – Consistency (Tutarlılık)**
    
    Her okuma en güncel veriyi görsün.
    
- **A – Availability (Kullanılabilirlik)**
    
    Sistem her zaman cevap verebilsin.
    
- **P – Partition Tolerance (Ağ bölünmesine dayanıklılık)**
    
    Network’te kopmalar olsa bile sistem bir şekilde çalışmaya devam edebilsin.
    

Teorem diyor ki:

**“Bir dağıtık sistemde aynı anda üçünü de tam sağlayamazsın.”**

Microservice dünyasında “P” zaten zorunlu (çok makine, çok network).

Dolayısıyla, durum şuna geliyor:

- Ya **daha çok C** diyorsun
- Ya **daha çok A** diyorsun

### Eventual Consistency (Sonunda tutarlılık)

Mikroservislerde genelde şunu kabul ederiz:

> Veriler her an %100 senkron olmak zorunda değil,
> 
> **kısa süre içinde tutarlı hale gelsin yeter.**

Örnek:

- Kullanıcı sipariş veriyor
- OrderService siparişi status = `Pending` ile kaydediyor
- PaymentService ödeme yapıyor → `Paid` event’i gönderiyor
- OrderService’te sipariş durumu `Paid` oluyor

Bu süreç 10–500 ms sürebilir → bu sırada biri sipariş detayına bakarsa eski durumu görebilir.

**Bu duruma “eventual consistency” deniyor.**

Monolit + tek DB’de genelde “strong consistency” var: transaction içinde hepsi bir anda değişiyor.

Mikroservis → “eventual consistency” ile yaşıyoruz;

SAGA, Outbox vs. bu dünyanın tasarım kalıpları.

---

## 0.5 – Mikroservisin Avantaj / Dezavantaj Özeti

### ✅ Avantajlar

- Bağımsız deploy (CI/CD kolay)
- Bağımsız ölçeklenme (ekonomik)
- Domain ayrımı (bounded context net)
- Farklı teknolojiler kullanabilme (Order .NET, Notification Node.js vb.)
- Takım organizasyonu kolay (ekip=servis)

### ❌ Dezavantajlar

- **Çok daha kompleks:**
    - Network, timeout, retry, circuit breaker
    - Logging, tracing (hangi request nereden geçti?)
    - Versiyon yönetimi
- **Dağıtık transaction yok** → SAGA, Outbox gibi pattern’ler şart
- **Monitoring & observability** daha zor
- Lokal geliştirme zor (10 servis + 5 tane altyapı servisiyle uğraşıyorsun)

> Kısa cümle:
> 
> Mikroservis, **problemin büyük ve karmaşık olduğu** noktada işine yarayan bir mimari.
> 
> Küçük CRUD proje için gereksiz.

---

## 0.6 – Ne Zaman Mikroservis Kullanmalıyım?

### Kullanılabilir (Mantıklı) Senaryolar

- Sistem çok büyük, tek monolit ekip ve teknoloji için zor
- Farklı feature’lar farklı hızda gelişiyor (Örn: Payment çok değişiyor, Notification sabit)
- Farklı tech stack ihtiyacı var (bazı kısımlar ML, bazıları klasik backend)
- Yük dağılımı farklı (Payment çok heavy, Order hafif)
- Takım sayısı fazla (5–10 ekip aynı anda çalışacak)

### Kullanılmaması Gereken Durumlar

- Küçük / orta boy ilk proje
- Tek ekip, az kişi
- Domain henüz oturmamış (ne neyi yapacak belli değil)
- Complex devops bilgisi yok

**Senin durumun:**

- Gerçek bir şirkette muhtemelen monolit + birkaç mikroservis karışımı göreceksin.
- Biz ise **öğrenmek için “abartılı ama mülakat için mükemmel” bir mikroservis projesi** yapıyoruz.

---

## 0.7 – Bizim Projede Bu Teori Ne Anlama Geliyor?

Toparlayalım ve bizim projeye bağlayalım:

- **Bounded Context**
    - Order, Payment, Inventory, Notification → ayrı domainler, ayrı servisler
- **Her servis ayrı DB**
    - OrderDb, PaymentDb, InventoryDb, NotificationDb (PostgreSQL)
- **Consistency**
    - Tek transaction yok → SAGA ile idare ediyoruz
    - Eventual consistency mantığını kabul ediyoruz
- **İletişim**
    - Başlangıçta REST kavramını gösterip
    - Sonra RabbitMQ ile event-driven’a geçiyoruz
- **Bağımsız deploy / scale**
    - Docker + (ileride Azure) ile her servisi bağımsız ayakta tutacağız

# 1. Mikroservisler Arası İletişim Neden Problem?

Mikroservis = bağımsız servisler

Ama gerçek dünyada hiçbir servis tek başına çalışmaz.

**Order → Payment**

**Payment → Inventory**

**Order → Notification**

her zaman bir konuşma ihtiyacı vardır.

Burada iki model var:

---

# 🟦 2. Senkron (Sync) İletişim – REST / HTTP

Bu klasik bildiğimiz yöntem:

```
OrderService → PaymentService'e POST çağrı atar

```

Örnek:

```
POST <http://payment/api/payments>
Body: { "orderId": 10, "amount": 250 }

```

Ve PaymentService’ten anında cevap bekler:

- 200 OK → ödeme başarılı
- 400 → eksik parametre
- 500 → ödeme sistemi çöktü

### Basit, tanıdık, öğrenmesi kolay.

Ama büyük bir PROBLEM var:

## ❌ Eğer PaymentService çökerse, OrderService de çöker

Neden?

Çünkü OrderService cevap bekliyor → Timeout → Hata → Tüm sipariş süreci çöker.

---

# 📌 2.1 Senkron Yapının Kötü Yanları

### ❌ 1. Servisler sıkı bağlı (coupled)

Order, Payment olmadan çalışamaz.

### ❌ 2. Downstream servis çökerse upstream de çöker

Payment down olunca Order down.

### ❌ 3. Yük arttığında sistem çökebilir

Örneğin kampanya sırasında:

- 10.000 sipariş geliyor
- PaymentService overload
- OrderService’in tüm threadleri bloklanır → sistem komple çöker

### ❌ 4. Yüksek latency

Her hop → network cost.

---

# 🟩 3. Asenkron (Async) İletişim – Event-Driven Architecture

Şimdi mikroservislerde **esas güç** buradan geliyor.

OrderService şunu yapmaz:

❌ “PaymentService kardeş, ödeme almanı istiyorum.”

Onun yerine:

✔ Bir event yayınlar: `OrderCreated`

✔ Bu event’i kimin dinlediği OrderService’in umurunda değildir.

✔ PaymentService dinliyorsa → işlem yapar

✔ Başka bir servis de dinleyebilir → o da işlem yapar

### Örnek Akış

```
OrderService → RabbitMQ → PaymentService → RabbitMQ → InventoryService

```

**OrderService sadece mesaj yayınlar.**

“Ödeme oldu mu?”, “Stok düştü mü?” diye REST çağrısı atmaz.

Bu, _loosely-coupled architecture_ demektir.

---

# ⭐ 3.1 Event-Driven Architecture’ın Avantajları

## ✔ 1. Loose Coupling (Bağımlılık çok azalır)

Order, Payment’in çalışıp çalışmadığını bilmez.

## ✔ 2. Servisler bağımsız ölçeklenebilir

PaymentService yoğun → 5 instance

OrderService sakin → 1 instance

Inventory → 2 instance

## ✔ 3. Daha dayanıklı sistem (fault-tolerant)

Payment down olsa bile:

- OrderService event'i kuyruğa yazar
- PaymentService gelince kaldığı yerden devam eder

## ✔ 4. Eventual consistency

Her şey hemen senkron güncellenmez, ama **bir süre sonra doğru hale gelir**.

Bu büyük sistemlerde doğal olarak kabul edilir.

## ✔ 5. Extended functionality

OrderCreated event’ini 10 farklı servis dinleyebilir:

- Payment
- Inventory
- Notification
- Analytics
- Fraud Detection (dolandırıcılık kontrolü)

OrderService hiçbirini bilmez → çok güçlü bir tasarım.

---

# 🐇 4. RabbitMQ Nedir?

RabbitMQ bir **message broker**’dır.

Görevi:

- Servislerin gönderdiği mesajları alır
- Queue’lara koyar
- Dinleyen servislere dağıtır

Yani:

```
Producer (OrderService)  -- event gönderir
RabbitMQ                 -- mesajı saklar/yönlendirir
Consumer (PaymentService) -- event tüketir

```

### RabbitMQ Bileşenleri (Basit Anlatım)

## 1) Exchange

Mesajların geldiği yer.

## 2) Queue

Mesajların tutulduğu yer.

## 3) Binding

Exchange → Queue bağlantısının kuralı.

## 4) Routing Key

Mesajın gideceği kuyruğu belirler.

---

# 🎯 5. RabbitMQ Türleri (En Çok Kullanılan)

## 1. **Fanout**

Event gelir → tüm queue’lara gönderilir

Yayın (broadcast) gibidir.

## 2. **Direct**

Routing key eşleşirse o queue’ya gider.

## 3. **Topic**

En güçlü:

`order.*`

`payment.#` gibi pattern’lerle routing yapılır.

# Aşama 0.3 — **SAGA Pattern Teorisi (Derin & Basit Anlatım)**

Bu konu, mikroservislerde **transaction** problemini çözer.

Bir kere öğrenince tüm taşlar yerine oturur.

---

# ❗ Önce Problemi Anlayalım:

## Monolith’te Transaction = Kolay

Monolit + tek DB’de:

```sql
BEGIN TRANSACTION
  Insert order
  Insert payment
  Update inventory
COMMIT

```

Eğer bir şey yanlış giderse:

```
ROLLBACK

```

Her şey geri alınır.

%100 **strong consistency**.

---

# ❗ Mikroserviste Transaction = İmkansız

Mikroserviste tablo artık üç farklı veritabanında:

- Order → OrderDb
- Payment → PaymentDb
- Inventory → InventoryDb

Artık şöyle bir şey **yapamazsın**:

```sql
BEGIN TRAN
  OrderDb insert
  PaymentDb insert
  InventoryDb update
COMMIT

```

**Neden?**

Çünkü her servis:

- Farklı DB
- Farklı makina
- Farklı network
- Farklı teknoloji (Postgres / Mongo / Redis olabilir)

Bu yüzden **distributed transaction** (2PC) mikroservislerde kullanılmaz.

Çok pahalı, çok yavaş, çok kırılgan.

Mikroservis dünyasında kabul edilen gerçek:

> Tek transaction yok. Her servis kendi transaction'ından sorumlu. İşlem zinciri eventual consistency ile sağlanır.

İşte SAGA bunun çözümüdür.

---

# 🎯 1. SAGA Pattern Nedir?

**SAGA = Dağıtık bir iş sürecini, bağımsız adımlar olarak yöneten mekanizma.**

Her adım kendi servisinde transaction açar:

1. OrderService → "OrderCreated" (transaction burada kapandı)
2. PaymentService → ödeme işlemi (transaction burada kapandı)
3. InventoryService → stok düşme işlemi (transaction burada kapandı)

Yani:

> Her adım kendi DB'sinde yapılır, bir sonraki adım event ile devam eder.

---

# 🎯 2. SAGA iki farklı yaklaşım içerir:

## 1) **Choreography (Dans eden servisler)**

Yaygın, RabbitMQ ile doğal şekilde yapılır.

Bizim projede öncelik bu.

## 2) **Orchestration (Orkestra şefi var)**

OrderSaga gibi bir merkez servis tüm adımları yönetir.

Her ikisini de sana anlatacağım.

---

# 🟦 1) CHOREOGRAPHY SAGA

**Hiçbir merkez yok. Servisler olayları dinler ve sıradaki adımı kendiliğinden başlatır.**

Tıpkı domino taşı gibi:

```
OrderCreated
    ↓
PaymentService: ödeme yap → PaymentSucceeded / PaymentFailed
    ↓
InventoryService: stok düş / iade et
    ↓
NotificationService: mail/sms at

```

Kimse “öbür servise git” demiyor.

Her servis sadece:

- Bir event yayınlıyor
- Başka bir event'i dinliyor

Tamamen gevşek bağlı (loosely coupled).

### Avantajları

- Basit
- En doğal mikroservis yaklaşımı
- Çok gevşek bağlı (loose coupling)
- Dağıtık yapıya çok uygun

### Dezavantajları

- İş akışı karmaşıksa anlaması zorlaşabilir
- Flow’u debug etmek daha zor

---

# 🟩 2) ORCHESTRATION SAGA

Bu modelde **bir yönetici servis** var:

```
OrderSaga Orchestrator

```

Akışı merkezi yönetir:

```
Orchestrator → PaymentService’e “ödeme yap”
PaymentService → “ok yaptım”
Orchestrator → InventoryService’e “stok düş”
InventoryService → “ok yaptım”
Orchestrator → NotificationService’e “mail at”

```

Tüm süreci tek bir servis kontrol eder.

### Avantajları

- Kritik akışlarda kontrol çok yüksek
- Debug etmek kolay
- İş akışı karmaşıksa daha düzenli

### Dezavantajları

- Orchestrator merkezi bir bağımlılık yaratır
- Tek yer bozulursa tüm süreç bozulur (single point of failure)
- Daha fazla kod ve yönetim yükü

Biz bu projede:

- İlk olarak **Choreography** modelini uygulayacağız
- Sonra istersen kısa bir Orchestrator versiyonu da gösterebilirim

---

# 💣 3. SAGA’da EN ÖNEMLİ KONU: **Compensation (Geri Alma)**

Diyelim ki sipariş akışı şöyle:

1. OrderCreated → başarı
2. PaymentSucceeded → başarı
3. InventoryReserved → **başarısız** (stok yok diyelim)

Şimdi ne olacak?

- Sipariş yaratıldı
- Ödeme alındı
- Stok YOK

Siparişi iptal etmek gerekiyor.

Ama payment zaten başarıyla tahsil edildi → geri iade etmelisin.

İşte buna SAGA’da:

> Compensating action denir.

Her servisin “reverse action”ı olmalı:

- PaymentService: `RefundPayment`
- OrderService: `CancelOrder`
- InventoryService: `ReleaseStock`

SAGA der ki:

**Eğer adım N fails → adım N-1, N-2, N-3 için geriye dönük telafi (compensation) işlemleri gönder.**

Biz projede bunu gerçek event'lerle yapacağız.

---

# 🔥 4. SAGA Yoksa Ne Olur? (Çok önemli)

SAGA yoksa:

- Sipariş açıldı
- Ödeme alındı
- Stok yok → Payment geri alınmadı → Müşteri mağdur
- Order status hâlâ “Created”
- Yarım kalmış siparişler

**Dağıtık sistemde 3 servis birden aynı anda garanti tutarlılık sağlayamaz. O yüzden SAGA şarttır.**

---

# 🎯 5. SAGA’yı Gerçek Sahnede Gösterelim (Bizim Projede)

Senaryomuz:

### ✔ 1. Adım → Sipariş Oluşturulur

OrderService:

- OrderDb’ye “Created” status’ü ile kaydeder
- RabbitMQ’ya **OrderCreated** event’i gönderir

### ✔ 2. Adım → PaymentService OrderCreated’ı dinler

Ödeme alır:

- Başarılıysa → PaymentSucceeded event
- Başarısızsa → PaymentFailed event

### ✔ 3. Adım → InventoryService PaymentSucceeded’ı dinler

Stok kontrol eder:

- Stok varsa → StockReserved event
- Yoksa → StockFailed event

### ✔ 4. Adım → OrderService final kararı verir

- StockReserved → Order status = Completed
- StockFailed → Order status = Cancelled + Payment refund event’i gönderir

### ✔ 5. Adım → PaymentService refund event’ini dinler

Ödemeyi iade eder

### ✔ 6. Adım → NotificationService tüm süreci loglar

---

# 🎉 6. SAGA’nın Efsane Özet Cümlesi (Mülakat İçin)

> "SAGA, mikroservislerde tek transaction yerine, her servisin kendi transaction'ını yapmasını ve adımların bir event zinciri içinde yönetilmesini sağlayan bir pattern’dir. Olumsuz bir durumda önceki adımları geri almak için compensation event’leri kullanılır.”

# Aşama 0.4 — **Outbox Pattern (Event Kaybolmaması İçin En Kritik Patern)**

Bu konu _kritik bir mülakat sorusu_,

üstelik gerçek hayatta en çok hata yapılan yer burasıdır.

Konuyu hem çok basit hem çok derin bir şekilde anlatacağım.

---

# 🎯 1. Outbox Pattern Neden Var? (Asıl Problem)

Event-Driven Architecture’da bir işlem yaptın diyelim:

## ✨ Order Service → “Sipariş oluştur”

1. OrderService veritabanına siparişi ekler
2. RabbitMQ’ya “OrderCreated” event’i gönderir

> Peki ikisini aynı anda garanti edebilir miyiz?

Hayır.

Burada iki farklı sistem var:

- PostgreSQL (DB)
- RabbitMQ (message broker)

Bu ikisi arasında **dağıtık transaction yoktur** (doğası gereği).

Ve bu bizi çok kritik 2 hataya götürür:

---

# ❌ 2. En Büyük Problem: “DB’ye yazdım ama event gitmedi”

Senaryoya bak:

```
Transaction:
  INSERT INTO Orders ...
Transaction COMMIT

RabbitMQ.Publish(event)

```

Eğer publish sırasında RabbitMQ down olursa:

✔ Order DB’ye kaydedildi

❌ Ama event Gitmedi

PaymentService bu siparişi **hiç duymamış olur**.

Bu bir **tutarsızlık (inconsistency)**.

Gerçek bir şirkette “sipariş var ama ödeme hiç tetiklenmedi” demektir.

---

# ❌ 3. Tersi: “Event gitti ama DB’ye yazılamadı”

```
RabbitMQ.Publish(event) (başarılı)
INSERT INTO Orders ... (DB çöktü, başarısız)

```

Bu da çok daha felaket:

- PaymentService event’i alıyor → ödeme yapıyor
- Ama OrderService’in kendi DB’sinde sipariş YOK !!!

Sistem “hayalet sipariş” üretiyor.

---

# ❌ 4. Bir başka risk: Double Publish (Mesaj iki kere gönderilir)

Eğer API çağrısı timeout olursa:

Client:

```
Retry POST /api/orders

```

OrderService:

- Siparişi tekrar oluşturur
- Event’i tekrar gönderir

PaymentService aynı siparişi 2 kere görür → **2 kere ödeme alabilir.**

Outbox Pattern bunun hepsini çözer.

---

# 🟩 5. Çözüm: Outbox Pattern

Outbox Pattern’in fikri çok basit ama çok zekice:

> DB’ye veri yazarken, aynı transaction içinde bir Outbox tablosuna event'i JSON olarak sakla. RabbitMQ’ya gönderme işini daha sonra, ayrı bir background işlem yapsın.

Yani:

## Doğru flow:

```
Transaction:
  Insert new order
  Insert new row into Outbox table   <-- event burada kaydedilir
COMMIT

Background worker:
  Read Outbox rows
  Publish to RabbitMQ
  Mark row as "Processed"

```

Bu ne sağlar?

### ✔ DB ve event yayınlama ASLA aynı transaction’da değildir

Ama **DB yazıldıysa, event kaybedilemez**.

### ✔ Event kaydı DB’de durduğu için

RabbitMQ down olsa bile event’ler kaybolmaz.

### ✔ Worker tekrar tekrar deneyebilir

Event gönderilemeyebilir → Worker 5 sn sonra tekrar dener.

### ✔ Duplicate publish engellenir

Outbox satırı işaretlenir (`Processed = true`).

### ✔ Exactly-once behavior’a çok yakın bir sonuç elde edilir

Mükemmel değil ama üretim ortamında kabul edilen yöntem.

---

# 🔎 6. Outbox Table Yapısı Nasıl Olur?

Örnek tablo:

|Id|EventType|Payload (json)|Status|CreatedAt|
|---|---|---|---|---|
|1|OrderCreated|"{...}"|Pending|2025-11-18|
|2|PaymentSucceeded|"{...}"|Pending|2025-11-18|

Status değerleri:

- Pending
- Processed
- Failed (retry için)

---

# 🔄 7. Background Worker Ne Yapıyor?

Worker her 5 saniyede bir:

1. `SELECT * FROM Outbox WHERE Status = 'Pending' LIMIT 50`
2. Her satırı RabbitMQ’ya publish eder
3. Publish başarılı olursa:
    - Status → Processed
4. Başarısız olursa:
    - Status → Pending bırakılır
    - Bir sonraki çalışmada tekrar denenir

Worker için iki seçenek var:

- HostedService (BackgroundService) → .NET içinde
- Ayrı bir Worker Service → bağımsız mikroservis

Biz ilk olarak HostedService ile başlayacağız, sonra gerçek ayrı worker’a da çevirebiliriz.

---

# 🎯 8. Outbox Pattern Neden Vazgeçilmez?

## ✨ 1. “At-least once” garantisi verir

Event tekrar da gidebilir ama ASLA kaybolmaz.

Bu üretimde en iyi çözümdür.

## ✨ 2. Dağıtık transaction problemine doğal çözüm

“Siparişi yazdım ama event gömdü” → Asla olmaz.

## ✨ 3. Retry mekanizması doğal olarak çalışır

RabbitMQ down → event'ler DB’de bekler.

## ✨ 4. Mülakatlarda en çok sorulan konudur

“Neden transaction içinden direkt event gönderilmez?”

“Outbox Pattern ne işe yarar?”

“Message gönderdikten sonra DB’ye yazılamazsa ne olur?”

---

# 🧠 9. Outbox Pattern – Mülakat Cevabı (Kısa, güçlü)

> “Mikroservislerde database ve message broker arasında distributed transaction kullanılamadığı için event kaybı veya double publish problemleri oluşur.
> 
> Outbox Pattern’de event, business işlemiyle aynı transaction içinde Outbox tablosuna kaydedilir.
> 
> Daha sonra bir background worker bu Outbox kayıtlarını message broker’a (RabbitMQ) yollar.
> 
> Bu sayede event kaybı engellenir ve dağıtık transaction ihtiyacı ortadan kalkar.”