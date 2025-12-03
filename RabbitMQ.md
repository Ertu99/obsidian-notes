#.NET Core Mikroservis Mimarilerinde RabbitMQ: Kapsamlı Mimari, Uygulama ve En İyi Pratikler Raporu

## 1. Giriş: Monolitikten Mikroservislere Geçiş ve Mesajlaşmanın Rolü

Yazılım geliştirme dünyasında son on yılda yaşanan en büyük paradigma değişimlerinden biri, monolitik mimarilerden dağıtık mikroservis mimarilerine geçiş olmuştur. Bir.NET Core geliştiricisi olarak bu yolculuğa yeni başladığınızda, karşılaşacağınız en temel zorluk "iletişim" olacaktır. Monolitik bir uygulamada, bir sipariş modülü stok modülüyle konuşmak istediğinde, aynı bellek bloğu (memory space) içinde basit bir metod çağrısı yapar. Bu işlem anlıktır, transactional bütünlük (ACID) veritabanı seviyesinde kolayca sağlanır ve ağ gecikmesi gibi bir dert yoktur.1

Ancak sistemi mikroservislere böldüğünüzde, "Sipariş Servisi" ve "Stok Servisi" artık farklı sunucularda, hatta farklı kıtalarda çalışıyor olabilir. Bu servislerin birbiriyle nasıl konuşacağı sorusu, sistemin başarısını belirleyen en kritik faktördür. İşte bu noktada, senkron (eşzamanlı) ve asenkron (eşzamanlı olmayan) iletişim arasındaki ayrım devreye girer. HTTP üzerinden REST API çağrıları yapmak (senkron iletişim), en yaygın başlangıç noktasıdır. Ancak, bir servis diğerinden yanıt beklerken kilitleniyorsa (blocking), zincirleme hatalar (cascading failures) ve performans darboğazları kaçınılmazdır.2

RabbitMQ gibi mesaj kuyruk sistemleri (Message Brokers), bu sorunu çözmek için "Asenkron Mesajlaşma" modelini sunar. RabbitMQ, üretici (Producer) ile tüketici (Consumer) arasına girerek onları hem **zamansal** (temporal) hem de **mekansal** (spatial) olarak birbirinden ayırır (decoupling). Bu rapor, yeni bir işe girmeyi hedefleyen ve.NET Core ile mikroservis dünyasına adım atan bir yazılım mühendisi için, RabbitMQ'nun en temel atomik parçalarından başlayarak, mülakatlarda sorulabilecek en karmaşık senaryolara kadar uzanan kapsamlı bir rehber niteliğindedir.

### 1.1 Mesaj Broker Nedir ve Neden RabbitMQ?

Mesaj Broker (Aracı), uygulamalar arasında veri alışverişini sağlayan bir ara yazılımdır. RabbitMQ, bu alandaki en olgun, en yaygın ve en güvenilir açık kaynaklı çözümlerden biridir. Erlang dili ile yazılmıştır; Erlang, telekomünikasyon sistemleri için tasarlanmış, yüksek eşzamanlılık (concurrency) ve hata toleransı (fault tolerance) sunan bir platformdur. RabbitMQ'nun bu temeli, onun milyonlarca mesajı düşük gecikme (latency) ile işlemesini sağlar.4

RabbitMQ'yu diğerlerinden (örneğin Kafka veya ActiveMQ) ayıran temel özellik, **AMQP 0-9-1 (Advanced Message Queuing Protocol)** standardını uyguluyor olmasıdır. RabbitMQ, "Akıllı Broker, Aptal Tüketici" (Smart Broker, Dumb Consumer) modelini benimser. Yani karmaşık yönlendirme mantığı (routing logic), mesajın hangi kuyruğa gideceği kararı ve mesajın teslim edildiğinin takibi Broker üzerinde yapılır. Bu, Kafka gibi "Akıllı Tüketici" modellerine göre, mikroservisler arasındaki karmaşık iş akışlarını yönetmeyi çok daha kolay hale getirir.2

Bir iş görüşmesinde "Neden RabbitMQ?" sorusuyla karşılaştığınızda vereceğiniz en güçlü cevap şudur: "RabbitMQ, karmaşık yönlendirme senaryolarını (routing), mesaj teslim garantilerini (reliability) ve servisler arası gevşek bağlılığı (loose coupling) standart bir protokol olan AMQP üzerinden en esnek şekilde sağlayan çözümdür.".2

---

## 2. AMQP 0-9-1 Protokolü ve Temel Kavramlar

RabbitMQ'yu anlamak, aslında AMQP 0-9-1 protokolünü anlamaktır. Bu protokol, sadece veri paketlerinin nasıl gönderileceğini değil, mesajlaşma sisteminin mimarisini de tanımlar. AMQP "Programlanabilir" bir protokoldür; yani kuyruklar, borsalar (exchanges) ve bağlamalar (bindings) gibi varlıklar, yönetici tarafından statik olarak tanımlanmak zorunda değildir; uygulamanız bu varlıkları kod (C#) üzerinden dinamik olarak oluşturabilir, değiştirebilir ve silebilir.5

### 2.1 Bağlantı (Connection) ve Kanal (Channel) Mimarisi

Yeni başlayanların ve hatta deneyimli geliştiricilerin en sık hata yaptığı, mülakatlarda en çok sorulan teknik detaylardan biri Bağlantı ve Kanal ayrımıdır. Bu ayrımı anlamak, performanslı bir.NET uygulaması yazmak için hayati öneme sahiptir.

#### Connection (TCP Bağlantısı)

RabbitMQ ile uygulamanız arasındaki fiziksel yoldur. Alt seviyede bir TCP bağlantısıdır. Bir TCP bağlantısı kurmak maliyetli bir iştir; "Three-way handshake" (üçlü el sıkışma), kimlik doğrulama (authentication) ve SSL/TLS el sıkışması gibi süreçleri içerir. İşletim sistemi seviyesinde de her TCP bağlantısı bir dosya tanımlayıcısı (file descriptor) tüketir. Mikroservis mimarisinde, saniyede binlerce mesaj işleyen yüzlerce thread (iş parçacığı) olabilir. Her işlem için yeni bir TCP bağlantısı açıp kapatmak, hem istemci makineyi hem de RabbitMQ sunucusunu "TCP Connection Churn" denilen duruma sokar ve sistemi kilitler.6

#### Channel (Sanal Bağlantı)

Bu sorunu çözmek için AMQP, "Channel" kavramını getirmiştir. Kanal, tek bir fiziksel TCP bağlantısı üzerinden geçen sanal bir iletişim yoludur (multiplexing). Tıpkı bir fiber optik kablo içinden geçen birden fazla ışık sinyali gibi, tek bir TCP bağlantısı üzerinden yüzlerce Kanal açabilirsiniz. Kanal açmak ve kapatmak çok ucuzdur ve ağ seviyesinde yeni bir el sıkışma gerektirmez.

|**Özellik**|**Connection**|**Channel**|
|---|---|---|
|**Doğa**|Fiziksel (TCP/IP)|Mantıksal (Sanal)|
|**Maliyet**|Yüksek (Handshake, Auth)|Çok Düşük|
|**Kullanım**|Uygulama ömrü boyunca 1 tane (Singleton)|İşlem başına veya Thread başına|
|**Thread Safety**|Thread-safe (Genellikle)|**Thread-safe DEĞİLDİR**|

**Kritik Uyarı:**.NET `RabbitMQ.Client` kütüphanesinde `IModel` (Channel) nesnesi thread-safe değildir. Asla aynı kanal nesnesini birden fazla thread arasında paylaştırmayın. Bu, paketlerin birbirine karışmasına ve protokol hatalarına neden olur. Doğru yöntem, `IConnection` nesnesini Singleton olarak tutmak, ancak her thread veya işlem için o bağlantıdan yeni bir kanal (`CreateModel`) türetmektir.6

---

## 3. RabbitMQ Çekirdek Bileşenleri: Exchange, Queue ve Binding

RabbitMQ'nun çalışma mantığını bir "Postane" analojisi ile zihninize kazıyabilirsiniz. Bir mektubu (Mesaj) posta kutusuna attığınızda, mektubun tam olarak hangi mahalledeki hangi eve (Kuyruk) gideceğini bilmezsiniz. Sadece zarfın üzerine bir adres (Routing Key) yazarsınız. Posta dağıtım merkezi (Exchange), bu adrese bakarak mektubu doğru posta kutusuna (Queue) yönlendirir.

RabbitMQ'da üretici (Producer) **ASLA** doğrudan bir kuyruğa mesaj göndermez. Mesajı her zaman bir Exchange'e gönderir. Bu kural, RabbitMQ'nun esnekliğinin temelidir.1

### 3.1 Exchange (Borsa/Dağıtıcı)

Exchange, mesajları üreticiden alır ve bunları belirli kurallara (Bindings) göre kuyruklara yönlendirir. Exchange bir depolama alanı değildir; sadece bir yönlendiricidir. Eğer bir mesaj bir Exchange'e gelir ve gideceği hiçbir kuyruk bulunamazsa, mesaj (konfigürasyona bağlı olarak) ya silinir ya da üreticiye geri iade edilir.11

Mülakatlarda karşınıza çıkacak en önemli soru: **"Exchange tipleri nelerdir ve aralarındaki farklar nedir?"**

#### 1. Direct Exchange (Doğrudan Yönlendirme)

En basit ve hedef odaklı yönlendirme tipidir. Mesajın üzerindeki `Routing Key` (Yönlendirme Anahtarı), kuyruğun `Binding Key` (Bağlama Anahtarı) ile **birebir** eşleşmelidir.

- **Senaryo:** Bir loglama sisteminiz var. `error` loglarını diskteki dosyaya yazmak istiyorsunuz. `info` loglarını ise sadece ekrana yazdırmak istiyorsunuz.
    
- **İşleyiş:** Üretici, mesajı `log_exchange` isimli direct exchange'e, `error` routing key'i ile gönderir. Bu exchange, `error` anahtarı ile kendine bağlanmış olan `DiskQueue` kuyruğuna mesajı iletir. `info` anahtarı ile gelen mesaj ise `DiskQueue`'ya gitmez.10
    
- **Default Exchange:** RabbitMQ'da her kuyruk, oluşturulduğu anda otomatik olarak "isimsiz" (empty string) bir Direct Exchange'e kendi ismiyle bağlanır. Bu sayede basitçe "Şu isimli kuyruğa gönder" dediğinizde aslında arka planda bu mekanizma çalışır.11
    

#### 2. Fanout Exchange (Yelpaze/Yayın Yönlendirme)

Bu tip, routing key'i tamamen görmezden gelir. Kendisine bağlı olan **tüm** kuyruklara mesajın bir kopyasını gönderir.

- **Senaryo (Pub/Sub):** Bir e-ticaret sitesinde yeni bir ürün eklendiğinde (`ProductCreated` olayı), hem "Arama Servisi"nin (Elasticsearch) indeksini güncellemesi, hem de "Mobil Bildirim Servisi"nin kullanıcılara bildirim atması gerekir.
    
- **İşleyiş:** Üretici mesajı `products_fanout` exchange'ine atar. Bu exchange'e bağlı `SearchQueue` ve `NotificationQueue` kuyruklarının ikisi de mesajı alır. Servisler birbirinden habersiz çalışır. Bu, mikroservislerdeki "gevşek bağlılık" (loose coupling) ilkesinin en güzel örneğidir.13
    

#### 3. Topic Exchange (Konu Tabanlı Yönlendirme)

En esnek ve güçlü yönlendirme tipidir. Routing key'ler genellikle nokta ile ayrılmış kelimelerden oluşur (örn: `stock.usd.nyse`, `stock.eur.frankfurt`). Eşleşme için joker karakterler (wildcards) kullanılır.

- **`*` (Yıldız):** Tam olarak bir kelimenin yerini tutar.
    
    - `stock.*.nyse` -> `stock.usd.nyse` ile eşleşir, ama `stock.usd.tech.nyse` ile eşleşmez.
        
- **`#` (Diyez/Hash):** Sıfır veya daha fazla kelimenin yerini tutar.
    
    - `stock.#` -> `stock` ile başlayan her şeyle eşleşir.
        
- **Senaryo:** Bir haber ajansı sisteminde, spor haberlerini, teknoloji haberlerini ve tüm "son dakika" (breaking) haberlerini ayrı ayrı filtrelemek istiyorsunuz. `news.sports.football`, `news.tech.ai`, `news.breaking.politics` gibi anahtarlar kullanabilirsiniz. Bir tüketici sadece `news.breaking.#` diyerek tüm son dakika haberlerini alabilir.4
    

#### 4. Headers Exchange

Routing key yerine, mesajın başlık (header) özelliklerine bakar. `x-match` argümanı ile `any` (herhangi biri eşleşirse) veya `all` (hepsi eşleşmeli) mantığı kurulabilir. Performansı Topic ve Direct exchange'lere göre daha düşüktür ve daha az kullanılır.4

### 3.2 Queues (Kuyruklar)

Kuyruklar mesajların tüketici tarafından işlenene kadar saklandığı tampon (buffer) bölgelerdir.

- **Durable (Dayanıklı):** RabbitMQ sunucusu yeniden başlatılsa bile kuyruk tanımı silinmez. (Dikkat: İçindeki mesajların silinmemesi için mesajın da 'Persistent' olması gerekir).
    
- **Transient (Geçici):** Sunucu kapanırsa kuyruk yok olur.
    
- **Auto-Delete:** Son tüketici (consumer) bağlantısını kestiğinde kuyruk otomatik olarak silinir.
    
- **Exclusive:** Sadece kuyruğu oluşturan bağlantı (connection) tarafından kullanılabilir ve bağlantı kapandığında silinir. Genellikle RPC senaryolarında geçici yanıt kuyrukları için kullanılır.5
    

### 3.3 Bindings (Bağlamalar)

Exchange ile Queue arasındaki kuraldır. "Bu Exchange'e gelen, şu Routing Key'e sahip mesajları, bu Kuyruğa ilet" talimatıdır. Yönlendirme mantığının kalbi burasıdır.4

---

## 4. Mesaj Güvenliği ve Dayanıklılık (Reliability)

Bir bankacılık uygulaması yazdığınızı düşünün. Bir para transferi mesajı kaybolursa ne olur? Kabul edilemez. RabbitMQ'da veri güvenliği, "Performance vs. Reliability" (Performans - Güvenilirlik) takasıdır. Mülakatlarda size "Mesajın kaybolmadığından nasıl emin olursun?" diye sorulacaktır. İşte cevabı:

### 4.1 Mesaj Kalıcılığı (Persistence)

Kuyruğu `Durable` yapmak yetmez. Mesajı üretirken `DeliveryMode` özelliğini `2` (Persistent) olarak ayarlamanız gerekir. Bu, RabbitMQ'nun mesajı RAM yerine diske yazmasını sağlar.

- **Maliyet:** Diske yazmak (I/O), RAM'e yazmaktan çok daha yavaştır. Performansı düşürür ama güvenliği artırır.17
    

### 4.2 Publisher Confirms (Yayıncı Onayları)

Mesajı socket.write ile ağa göndermeniz, mesajın RabbitMQ'ya ulaştığı anlamına gelmez. Ağ kopabilir. RabbitMQ, mesajı alıp güvenli bir şekilde (diske veya kuyruğa) kaydettiğinde yayıncıya bir Ack (Onay) sinyali gönderir.

.NET Core'da bunu kullanmak için kanalda ConfirmSelect aktif edilmelidir. WaitForConfirmsOrDie metodu ile senkron bekleyebilir (yavaş) veya olay tabanlı (asenkron) callbacks kullanabilirsiniz.20

### 4.3 Consumer Acknowledgements (Tüketici Onayları)

RabbitMQ, bir mesajı tüketiciye gönderdiğinde, o mesajı kuyruktan ne zaman silmeli?

- **Auto Ack (Otomatik Onay):** RabbitMQ mesajı ağa gönderdiği an siler. Tüketici mesajı alıp işlerken elektrik kesilirse veya kod hata verirse (exception), mesaj kaybolur. Bu hızlıdır ama risklidir.22
    
- **Manual Ack (Elle Onay):** Tüketici mesajı alır, işler (veritabanına yazar) ve _sonra_ RabbitMQ'ya "Ben işimi bitirdim, silebilirsin" (`BasicAck`) der. Eğer tüketici bu onayı göndermeden ölürse (bağlantı koparsa), RabbitMQ mesajın işlenmediğini anlar ve başka bir tüketiciye tekrar gönderir. "At-Least-Once Delivery" (En az bir kere teslimat) garantisi bu şekilde sağlanır.23
    

**Best Practice:** İş kritik sistemlerde daima Manual Ack kullanın. Ack işlemini `finally` bloğunda değil, işlem _başarıyla_ bittikten sonra yapın.

---

## 5..NET Core ile Uygulama ve Kodlama Pratikleri

Teoriyi öğrendik, şimdi.NET Core dünyasına inelim. Projenizde `RabbitMQ.Client` NuGet paketini kullanacaksınız. Ancak modern.NET dünyasında, MassTransit gibi soyutlama kütüphaneleri daha yaygındır (buna 7. bölümde değineceğiz). Önce "native" (yerel) istemciyi anlamak, temel için şarttır.

### 5.1 Dependency Injection ve Servis Yaşam Döngüsü

.NET Core'da RabbitMQ bağlantısını yönetmek için en doğru yöntem **Singleton** tasarım desenidir.

C#

```
// IRabbitMqConnection.cs
public interface IRabbitMqConnection
{
    IConnection Connection { get; }
}

// RabbitMqConnection.cs
public class RabbitMqConnection : IRabbitMqConnection, IDisposable
{
    private readonly IConnection _connection;
    public RabbitMqConnection(IConfiguration configuration) 
    {
        var factory = new ConnectionFactory() 
        { 
            HostName = configuration,
            // Otomatik yeniden bağlanma (network recovery) çok önemlidir
            AutomaticRecoveryEnabled = true 
        };
        _connection = factory.CreateConnection();
    }
    //... Dispose mantığı
}
```

Bu servisi `Startup.cs` veya `Program.cs` içinde `services.AddSingleton<IRabbitMqConnection, RabbitMqConnection>();` olarak eklemelisiniz.25

### 5.2 Producer (Üretici) Kodlama

Üretici tarafında mesaj yayınlarken dikkat etmeniz gereken en önemli nokta, kanalı (`IModel`) her işlem için oluşturup `using` bloğu ile kapatmaktır (veya bir kanal havuzu kullanmaktır).

C#

```
public void SendMessage(string message)
{
    // Bağlantıyı Singleton servisten alıyoruz
    using var channel = _rabbitMqConnection.Connection.CreateModel();
    
    // Exchange'i her zaman declare etmek (varsa dokunmaz, yoksa oluşturur) iyi bir pratiktir (Idempotency)
    channel.ExchangeDeclare("siparis_exchange", ExchangeType.Direct, durable: true);

    var body = Encoding.UTF8.GetBytes(message);
    var properties = channel.CreateBasicProperties();
    properties.Persistent = true; // Mesajı diske yaz!

    channel.BasicPublish(exchange: "siparis_exchange",
                         routingKey: "siparis.yeni",
                         basicProperties: properties,
                         body: body);
}
```

.27

### 5.3 Consumer (Tüketici) ve BackgroundService

Tüketiciler genellikle kullanıcı isteğinden bağımsız, arka planda sürekli çalışan servislerdir..NET Core'da bunun karşılığı `BackgroundService` veya `IHostedService`'tir.

C#

```
public class OrderProcessingWorker : BackgroundService
{
    private IModel _channel;
    private readonly IConnection _connection;

    public OrderProcessingWorker(IRabbitMqConnection connectionService)
    {
        _connection = connectionService.Connection;
    }

    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _channel = _connection.CreateModel();
        _channel.QueueDeclare("siparis_kuyrugu", durable: true, exclusive: false, autoDelete: false);
        _channel.QueueBind("siparis_kuyrugu", "siparis_exchange", "siparis.yeni");

        // QoS (Quality of Service) - Prefetch Count
        // Bu ayar hayati önem taşır. Tüketiciye aynı anda sadece 1 mesaj ver.
        // O mesaj onaylanmadan (Ack) yenisini verme. Yük dengelemesi sağlar.
        _channel.BasicQos(0, 1, false);

        var consumer = new EventingBasicConsumer(_channel);
        consumer.Received += (model, ea) =>
        {
            var body = ea.Body.ToArray();
            var message = Encoding.UTF8.GetString(body);
            
            try
            {
                ProcessOrder(message); // İş mantığı
                
                // Başarılı ise onayla
                _channel.BasicAck(ea.DeliveryTag, multiple: false);
            }
            catch (Exception)
            {
                // Hata varsa reddet. Requeue=false derseniz DLX'e gider, true derseniz döngüye girer.
                _channel.BasicNack(ea.DeliveryTag, multiple: false, requeue: false);
            }
        };

        _channel.BasicConsume(queue: "siparis_kuyrugu", autoAck: false, consumer: consumer);
        return Task.CompletedTask;
    }
}
```

Buradaki `BasicQos(prefetchCount: 1)` ayarı, "Fair Dispatch" (Adil Dağıtım) sağlar. Eğer bu ayarı yapmazsanız, RabbitMQ kuyruktaki tüm mesajları bir anda tüketiciye yığabilir (Round Robin), bu da belleğin şişmesine ve yük dengesizliğine yol açar.

RabbitMQ'da **QoS (Quality of Service)**, genellikle **"Basic QoS"** veya **"Prefetch Count"** ayarı olarak bilinir. Bu mekanizma, mesajların kuyruktan (queue) tüketicilere (consumers) nasıl dağıtılacağını kontrol etmenizi sağlar.

Temel amacı: **Tüketicinin kapasitesini aşmayacak şekilde, adil bir iş dağılımı (Fair Dispatch) yapmaktır.**

İşte RabbitMQ QoS'in ne olduğu ve neden kritik öneme sahip olduğunun detayları:

---

### 1. Varsayılan Davranış (Round-robin)

QoS ayarı yapılmadığında, RabbitMQ varsayılan olarak mesajları tüketicilere sırayla (Round-robin) dağıtır.

- **Senaryo:** Kuyrukta 100 mesaj var ve 2 tüketiciniz (Consumer A ve Consumer B) var.
    
- **Davranış:** RabbitMQ, tüketicilerin o an meşgul olup olmadığına bakmaksızın, 1. mesajı A'ya, 2. mesajı B'ye, 3. mesajı A'ya gönderir.
    
- **Sorun:** Eğer tek sayılardaki mesajlar çok ağır işlemler (örn: video işleme) ve çift sayılardaki mesajlar çok hafif işlemler (örn: e-posta atma) ise; Consumer A işlerin altında ezilirken, Consumer B işini hemen bitirip boş boş oturacaktır.
    

### 2. QoS ve Prefetch Count Çözümü (Fair Dispatch)

QoS ayarını kullanarak RabbitMQ'ya şunu söylersiniz: **"Bir tüketici elindeki işi bitirip onay (acknowledge) verene kadar ona yeni bir mesaj gönderme."**

Bunu `prefetch_count` değeri ile belirlersiniz.

- **prefetch_count = 1:** Tüketiciye aynı anda sadece 1 mesaj verilir. O mesaj işlenip `Ack` (onay) gelmeden 2. mesaj gönderilmez.
    
- **Sonuç:** Ağır işi yapan Consumer A meşgulken, RabbitMQ yeni gelen mesajları boşta olan Consumer B'ye yönlendirir. Böylece yük dengelenmiş olur.
    

### 3. Nasıl Kullanılır? (C# Örneği)

Bu ayarın çalışabilmesi için **AutoAck** (Otomatik Onay) özelliğinin **kapalı** olması gerekir. Mesaj onayını sizin kodunuzun (manuel olarak) vermesi gerekir.

C#

```
var factory = new ConnectionFactory() { HostName = "localhost" };
using (var connection = factory.CreateConnection())
using (var channel = connection.CreateModel())
{
    // ... Queue tanımlamaları ...

    // QoS AYARI BURADA YAPILIR
    // prefetchSize: 0 (Genellikle 0 bırakılır, mesaj boyutu sınırı yok demektir)
    // prefetchCount: 1 (Tüketici aynı anda sadece 1 mesaj işlesin)
    // global: false (Bu ayar sadece bu channel üzerindeki bu tüketici için geçerli olsun)
    
    channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);

    var consumer = new EventingBasicConsumer(channel);
    
    consumer.Received += (model, ea) =>
    {
        var body = ea.Body.ToArray();
        
        // İşlemlerinizi burada yaparsınız (Örn: Veritabanı kaydı)
        DoHeavyWork(); 

        // İŞLEM BİTTİKTEN SONRA ONAY GÖNDERİLİR
        // Bu onay gidene kadar RabbitMQ bu tüketiciye yeni mesaj göndermez.
        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
    };

    // AutoAck = false olmalıdır!
    channel.BasicConsume(queue: "task_queue", autoAck: false, consumer: consumer);
}
```

### 4. Parametrelerin Anlamları

RabbitMQ'da `BasicQos` metodu genellikle üç parametre alır:

1. **prefetchSize (uint):**
    
    - Tüketiciye gönderilecek, henüz onaylanmamış mesajların toplam boyutunu (byte cinsinden) sınırlar.
        
    - Genellikle **0** verilir (sınırsız), çünkü mesaj boyutu yerine mesaj adediyle yönetmek daha kolaydır.
        
2. **prefetchCount (ushort):**
    
    - En kritik ayardır. Tüketicinin aynı anda işleyebileceği, henüz onaylanmamış (unacked) maksimum mesaj sayısını belirtir.
        
    - `1` yaparsanız: "Birini bitirmeden yenisini atma" demektir.
        
3. **global (bool):**
    
    - `false`: Bu limit, o anki _yeni_ tüketici (consumer) bazında uygulanır.
        
    - `true`: Bu limit, o `channel` üzerindeki _tüm_ tüketiciler için toplam limit olarak uygulanır.
        

### Özetle Neden Kullanmalısın?

1. **Yük Dengeleme:** Hızlı ve yavaş tüketiciler arasında adaleti sağlar.
    
2. **Bellek Yönetimi:** Tüketicinin belleğinin binlerce işlenmemiş mesajla dolmasını engeller.
    
3. **Hata Yönetimi:** Tüketici çökerse, sadece elindeki (henüz onaylanmamış) 1 mesaj kaybolmaz, RabbitMQ o mesajı başka bir tüketiciye yönlendirir.
    


---

## 6. İleri Seviye Konseptler ve Mülakat Soruları

Deneyimli bir aday olarak öne çıkmanızı sağlayacak konular buradadır.

### 6.1 Dead Letter Exchange (DLX) ve "Poison Message" Yönetimi

Bir mesaj, tüketicinin kodundaki bir bug veya verideki bir bozukluk nedeniyle işlenemiyorsa ne olur? Eğer requeue=true ile sürekli reddederseniz, mesaj sonsuz bir döngüye girer (RabbitMQ -> Consumer -> Hata -> RabbitMQ). Bu mesajlara "Zehirli Mesaj" (Poison Message) denir.

Çözüm: Kuyruğu tanımlarken bir DLX (Ölü Mektup Borsası) belirtmektir. Hata alan veya süresi dolan (TTL) mesajlar otomatik olarak bu Exchange'e, oradan da bir "Hata Kuyruğu"na yönlendirilir. Geliştiriciler daha sonra bu kuyruğu inceleyerek hatanın nedenini anlar.31

### 6.2 RPC (Remote Procedure Call)

RabbitMQ asenkrondur ama bazen bir servisin diğerinden cevaba ihtiyacı olur. RPC pattern'i burada devreye girer.

1. İstemci (Client) bir istek mesajı gönderir ve içinde `ReplyTo` (Cevap nereye dönecek?) ve `CorrelationId` (Bu cevap hangi isteğin?) özelliklerini set eder.
    
2. Sunucu (Server) işlemi yapar ve sonucu `ReplyTo` kuyruğuna, aynı `CorrelationId` ile yazar.
    
3. İstemci, cevap kuyruğunu dinler ve ID eşleşmesi yaparak cevabı alır.
    
    Bu yapı, request/response mimarisini mesajlaşma üzerinden simüle eder.34
    

### 6.3 Kümeleme (Clustering) ve Quorum Queues

Tek bir RabbitMQ sunucusu (Node) çökerse sistem durur. Prod ortamında mutlaka Cluster (Küme) kurulmalıdır.

Eskiden "Mirrored Queues" (Aynalanmış Kuyruklar) kullanılırdı ama artık Quorum Queues standarttır. Raft konsensüs algoritmasını kullanan Quorum kuyruklar, verinin birden fazla sunucuda tutarlı bir şekilde saklanmasını sağlar ve ağ bölünmelerine (network partitions) karşı çok daha dirençlidir.33

---

## 7. MassTransit: Neden Native Client Yerine Kullanmalıyım?

İş ilanlarında sıkça "MassTransit" veya "NServiceBus" görürsünüz. Native `RabbitMQ.Client` ile çalışmak, TCP soketleri ile uğraşmaya benzer; çok fazla "boilerplate" (basmakalıp) kod yazarsınız. Retry mekanizması, serileştirme (JSON/XML), DLX yönetimi gibi işleri elle kodlamanız gerekir.

MassTransit,.NET için geliştirilmiş, RabbitMQ'nun üzerine oturan bir soyutlama katmanıdır (Abstraction Layer).

- **Kolaylık:** `ExchangeDeclare`, `QueueBind` gibi topoloji tanımlarını sizin yerinize otomatik yapar.
    
- **Resilience (Direnç):** İçinde gömülü Retry (Tekrar deneme), Circuit Breaker (Devre kesici) patternleri ile gelir. Örneğin "Hata alırsan 3 kere dene, her denemede 5 saniye bekle" gibi kuralları tek satırla eklersiniz.36
    
- **Sagas (State Machines):** Dağıtık sistemlerde "Transaction" yönetimi zordur. MassTransit, uzun süreli iş akışlarını (Saga) yönetmek için harika bir State Machine desteği sunar.37
    

**Mülakat İpucu:** "Native client mı MassTransit mi?" sorusuna, "Basit, tek yönlü işler ve maksimum performans/kontrol gerektiren durumlar için Native Client; ancak kurumsal, karmaşık iş akışları, hata yönetimi ve Sagas gerektiren mikroservis mimarileri için MassTransit kullanırım çünkü tekerleği yeniden icat etmeyi engeller ve geliştirme hızını artırır" şeklinde cevap verin.37

---

## 8. Üretim Ortamı (Production) Kontrol Listesi

Yeni işinizde sistemi canlıya almadan önce şunları kontrol etmelisiniz:

|**Kontrol Noktası**|**Detay**|**Neden?**|
|---|---|---|
|**Kullanıcılar**|`guest` kullanıcısını silin veya kısıtlayın.|Güvenlik açığıdır.|
|**Resource Limits**|Memory High Watermark ayarını kontrol edin.|RAM dolarsa RabbitMQ tüm yayıncıları bloklar.|
|**Alarmlar**|Disk alanı ve RAM için Prometheus/Grafana alarmları kurun.|Sistem çökmeden haberiniz olmalı.|
|**Dosya Tanımlayıcılar**|OS seviyesinde `ulimit -n` değerini artırın (örn: 65536).|Çok fazla bağlantı açıldığında sistem hata vermesin.|
|**VHost**|Farklı uygulamalar için farklı Virtual Host'lar kullanın.|İzolasyon ve güvenlik sağlar.|

.39

---

## 9. Sonuç

RabbitMQ,.NET Core mikroservis dünyasında sadece bir araç değil, mimarinin omurgasıdır. Bu raporda ele aldığımız AMQP protokol detayları, Exchange-Queue mekanizmaları ve güvenilirlik desenleri, sizi sadece bir "kullanıcı" olmaktan çıkarıp, sistemin "nasıl ve neden" çalıştığını bilen bir mühendis seviyesine taşıyacaktır.

Yeni işinizde başarılar dilerim; unutmayın, detaylar (Ack mekanizması, Thread güvenliği, QoS ayarları) profesyonelleri amatörlerden ayıran ince çizgilerdir. Bu raporu bir başvuru kaynağı olarak kullanabilir, takıldığınız noktalarda ilgili bölüme dönerek bilgilerinizi tazeleyebilirsiniz.