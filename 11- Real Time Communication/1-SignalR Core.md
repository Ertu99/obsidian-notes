
Şimdi .NET dünyasının gerçek zamanlı iletişim standardı olan, Microsoft'un yeniden yazdığı ve hafiflettiği **SignalR Core** kütüphanesini, sadece "Chat yapma aracı" olmaktan çıkarıp, **RPC (Remote Procedure Call)** mimarisi ve **Scale-out (Yatay Büyüme)** stratejileriyle derinlemesine inceleyelim.

# SignalR Core: Derinlemesine Mimari

SignalR Core, sunucu ve istemci arasındaki karmaşık el sıkışma, bağlantı yönetimi ve veri serileştirme işlerini soyutlayan (abstract) bir **RPC Framework**'üdür.

Sen kod yazarken "WebSocket paketi oluştur" demezsin; "İstemcideki `ReceiveMessage` fonksiyonunu çağır" dersin. Aradaki büyüyü SignalR yapar.

### 1. Hub (Merkez) Mimarisi

SignalR'ın kalbi **Hub** sınıfıdır. Burası, tüm trafiğin yönetildiği kule gibidir.

C#

```cs
// Sunucu Tarafı (C#)
public class ChatHub : Hub
{
    // İstemciler bu metodu tetikler (RPC)
    public async Task SendMessage(string user, string message)
    {
        // Sunucu, bağlı olan TÜM istemcilerin 'ReceiveMessage' metodunu tetikler
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

**Mühendislik Detayı (Strongly Typed Hubs):** `SendAsync("ReceiveMessage", ...)` demek **Magic String** kullanmaktır ve hataya açıktır. Profesyonel projelerde **`Hub<T>`** kullanılır.

C#

```cs
public interface IChatClient
{
    Task ReceiveMessage(string user, string message);
}

// Artık derleme zamanı (Compile Time) güvenliği var!
public class TypedChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.ReceiveMessage(user, message);
    }
}
```

### 2. Transport Fallback (Taşıma Yedeği) Mekanizması

SignalR'ın en büyük gücü "Her yerde çalışması"dır. Bağlantı kurarken sırasıyla şunları dener (Graceful Degradation):

1. **WebSockets:** En iyisi. Çift yönlü, düşük gecikme.
    
2. **Server-Sent Events (SSE):** Eğer WebSocket engelliyse (Kurumsal Firewall/Proxy) buna geçer.
    
3. **Long Polling:** Hiçbiri çalışmazsa buna düşer.
    

**Mühendislik Ayarı:** Eğer uygulamanın sadece modern tarayıcılarda çalışacağını biliyorsan, performans kazanmak için `SkipNegotiation = true` diyerek doğrudan WebSocket'e zorlayabilirsin (Sticky Session ihtiyacını da ortadan kaldırır, ancak sadece WebSocket açıksa çalışır).

### 3. Protokol: JSON vs MessagePack

SignalR varsayılan olarak veriyi **JSON** formatında gönderir.

- _Avantaj:_ Okunabilir, debug etmesi kolay.
    
- _Dezavantaj:_ Metin tabanlı olduğu için boyut büyüktür ve CPU yorar.
    

**Performans Mühendisliği (Binary Protocol):** Yüksek trafikli (High Frequency Trading, Oyun) sistemlerde **MessagePack** protokolü açılır.

- JSON'a göre %70 daha küçüktür.
    
- Binary olduğu için çok daha hızlı parse edilir.
    
- _Kod:_ `builder.Services.AddSignalR().AddMessagePackProtocol();`
    

### 4. Bağlantı Yönetimi: ConnectionId, Users ve Groups

Hub içinde `Context.ConnectionId` her bağlantı için (F5'e basınca değişen) benzersiz bir GUID'dir. Ancak iş mantığında bu yetmez.

1. **Users:** SignalR, `IUserIdProvider` kullanarak o anki bağlantının hangi kullanıcıya (Identity Name / Subject ID) ait olduğunu bilir.
    
    - `Clients.User("ahmet@mail.com").SendAsync(...)` diyerek, Ahmet'in o an açık olan tüm cihazlarına (Telefon, Laptop, Tablet) aynı anda mesaj atabilirsin.
        
2. **Groups:** En güçlü özelliktir. Bir bağlantıyı bir odaya sokarsın.
    
    - `Groups.AddToGroupAsync(Context.ConnectionId, "PremiumUsers")`
        
    - `Clients.Group("PremiumUsers").SendAsync(...)`
        

### 5. Hub Dışından Mesaj Göndermek (`IHubContext`)

Hub sınıfı, sadece istemciden bir istek geldiğinde (Transient) yaşar. Peki, arka planda çalışan bir `BackgroundWorker` veya bir `Controller` içinden (HTTP POST geldiğinde) kullanıcılara nasıl mesaj atarsın?

**Çözüm: `IHubContext<ChatHub>`** Bu arayüzü DI Container'dan enjekte edersin.

C#

```cs
[ApiController]
public class NotificationController : ControllerBase
{
    private readonly IHubContext<ChatHub> _hubContext;

    public NotificationController(IHubContext<ChatHub> hubContext)
    {
        _hubContext = hubContext;
    }

    [HttpPost("alert")]
    public async Task SendAlert(string msg)
    {
        // Hub instance'ı yaratmadan dışarıdan mesaj atar
        await _hubContext.Clients.All.SendAsync("ReceiveAlert", msg);
    }
}
```

### 6. Ölçeklenme: Redis Backplane ve Azure SignalR Service

Bir önceki derste Redis Backplane mantığını konuşmuştuk. Ancak Microsoft'un bir de "Parayı ver, derdi unut" çözümü vardır: **Azure SignalR Service**.

- **Backplane (Redis):** Sunucuları yönetmek, Redis'i ayakta tutmak, Sticky Session ayarları senin sorumluluğundadır.
    
- **Azure SignalR Service (PaaS):**
    
    1. Senin sunucun (App Server) ile İstemci (Browser) arasında bağlantı yoktur.
        
    2. İstemci doğrudan Azure'a bağlanır.
        
    3. Senin sunucun sadece Azure'a "Şunlara mesaj at" der.
        
    4. **Fayda:** Sunucun 1 milyon bağlantıyı (Socket) yönetmek zorunda kalmaz. RAM tasarrufu sağlar ve sonsuz ölçeklenir.