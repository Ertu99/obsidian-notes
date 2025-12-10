
Şimdi RabbitMQ gibi "ham" motorları kullanmayı çok daha kolay, güvenli ve standart hale getiren, .NET dünyasının Enterprise Service Bus (ESB) şampiyonu **MassTransit**'e geçiyoruz.

MassTransit, RabbitMQ'nun (veya Azure Service Bus'ın) üzerine giydirilmiş **çelik bir zırhtır.** Ham sürücülerle uğraşırken yaşayacağın bağlantı kopması, serileştirme hataları, DI scope yönetimi gibi dertleri senin yerine çözer.

Bu konuyu; **Abstraction (Soyutlama)**, **Saga (State Machine)** ve **Outbox Pattern** gibi çok ileri seviye mimari desenler üzerinden inceleyelim.

---

### 1. Felsefe: "Transport Agnostic" (Taşıma Aracı Bağımsızlığı)

MassTransit'in en büyük vaadi şudur: **"Kodunu yaz, altyapıyı sonra seç."**

Bugün RabbitMQ kullanıyorsun. Yarın Azure Service Bus'a, öbür gün Amazon SQS'e geçmen gerekti.

- **Ham Kütüphane (`RabbitMQ.Client`):** Tüm kodunu çöpe atıp yeniden yazman gerekir.
    
- **MassTransit:** Sadece `Program.cs` içindeki bir satırı değiştirirsin (`.UsingRabbitMq` -> `.UsingAzureServiceBus`). Tüketici (Consumer) kodların, event sınıfların, iş mantığın **tek satır bile değişmez.**
    

---

### 2. Mimari: Consumer Pattern ve DI Yönetimi

RabbitMQ'da `BasicConsumer` sınıfını implemente etmek ve byte array parse etmek zordur. MassTransit'te ise her şey **Tip Güvenli (Strongly Typed)** sınıflardır.

C#

```cs
// Consumer Tanımı
public class OrderCreatedConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        var orderId = context.Message.OrderId;
        // İş mantığı...
    }
}
```

Mühendislik Detayı (Scope Management):

RabbitMQ'nun kendi kütüphanesinde Dependency Injection (Scoped Service) kullanmak tam bir baş belasıdır (Scope'u elle açıp kapatman gerekir).

MassTransit bunu otomatik yapar. Consume metodu çalıştığında senin için bir Scope yaratır, iş bitince Dispose eder. DbContext'i güvenle enjekte edebilirsin.

---

### 3. Mesaj Tipleri: Command vs Event

MassTransit bu iki kavramı net bir şekilde ayırır:

1. **Command (Komut):** Bir şeyi **yapmasını** istersin.
    
    - `Send` metodu kullanılır.
        
    - Hedef bellidir (Tek bir kuyruğa gider).
        
    - Örn: `Send(new CreateInvoice { ... })` -> Sadece Fatura Servisine gider.
        
2. **Event (Olay):** Bir şeyin **olduğunu** bildirirsin.
    
    - `Publish` metodu kullanılır.
        
    - Hedef belli değildir (Pub/Sub). Kim dinliyorsa ona gider.
        
    - Örn: `Publish(new UserRegistered { ... })` -> Mail servisi de, Rapor servisi de, Hoşgeldin servisi de bunu duyar.
        

---

### 4. İleri Seviye: Request/Response Pattern (RPC)

Mesajlaşma asenkrondur dedik. Peki ya cevaba ihtiyacın varsa?

Örneğin: "Stok var mı?" diye sordun, "Var/Yok" cevabını almadan siparişi tamamlayamıyorsun.

MassTransit, asenkron üzerinden senkron bir illüzyon yaratır.

C#

```cs
// İstemci
var response = await _client.GetResponse<StockStatus>(new CheckStock { Id = 1 });

if (response.Message.IsAvailable) { ... }
```

Nasıl Çalışır?

MassTransit arka planda geçici bir kuyruk (Temporary Queue) oluşturur. İsteği gönderirken "Cevabı şu geçici kuyruğa at" der. Cevap gelene kadar await ile bekler. Cevap gelince o geçici kuyruğu siler.

---

### 5. En Güçlü Silah: Saga State Machine

Mikroservislerde "Distributed Transaction" yönetmek için **Saga** kullanılır demiştik. MassTransit, **Automatonymous** kütüphanesiyle görsel bir **State Machine (Durum Makinesi)** sunar.

Sipariş sürecini kodla çizersin:

C#

```cs
// Siparişin Durum Makinesi
public class OrderStateMachine : MassTransitStateMachine<OrderState>
{
    public OrderStateMachine()
    {
        // Başlangıç: Sipariş Geldi
        Initially(
            When(OrderSubmitted)
                .Then(context => context.Instance.CreatedDate = DateTime.Now)
                .TransitionTo(Submitted) // Durumu değiştir
                .Publish(context => new TakePayment { ... }) // Ödeme servisine emir ver
        );

        // Ödeme Başarılıysa -> Stok düş
        During(Submitted,
            When(PaymentAccepted)
                .TransitionTo(Paid)
                .Publish(context => new ReserveStock { ... })
        );
        
        // Ödeme Başarısızsa -> İptal et (Compensating Transaction)
        During(Submitted,
            When(PaymentFailed)
                .TransitionTo(Cancelled)
                .Publish(context => new CancelOrder { ... })
        );
    }
}
```

---

### 6. Kritik Desen: The Outbox Pattern (Veri Tutarlılığı)

Bir Mimarın bilmesi gereken en önemli desenlerden biridir.

**Sorun (Dual Write Problem):**

C#

```cs
// 1. Veritabanına kaydet
_dbContext.Orders.Add(order);
await _dbContext.SaveChangesAsync();

// 2. Mesaj kuyruğuna at
await _bus.Publish(new OrderCreated { ... });
```

Ya 1. adım başarılı olur da (DB'ye yazıldı), tam 2. adıma geçerken sunucu çökerse?

Veritabanında sipariş var ama kuyrukta mesaj yok. Diğer servislerin haberi olmadı. Veri tutarsızlığı (Inconsistency) oluştu.

Çözüm: MassTransit Outbox

MassTransit, Entity Framework Core ile entegre çalışır.

1. Sen `Publish` dediğinde, mesajı RabbitMQ'ya göndermez.
    
2. Mesajı senin veritabanındaki özel bir tabloya (`OutboxMessage`) kaydeder.
    
3. `SaveChangesAsync` dediğinde hem sipariş hem de mesaj **aynı Transaction içinde** (Atomik olarak) veritabanına yazılır.
    
4. Arka plandaki bir servis, veritabanındaki o mesajı okur ve RabbitMQ'ya gönderir.
    

Bu sayede **%100 Tutarlılık** sağlanır.

---
