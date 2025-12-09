 İstemci (Client) her şeyi sunucudan istediği (Pull), sunucunun pasif olduğu klasik HTTP dünyasından çıkıyoruz. Şimdi sunucunun inisiyatif alıp "Al sana veri!" diyebildiği (Push), canlı ve yaşayan sistemlere, **Real-Time Communication (Gerçek Zamanlı İletişim)** dünyasına giriyoruz.

Bu konuyu sadece "Chat uygulaması yapmak" olarak görme. Borsadaki fiyat akışından, Uber'deki taksi konumuna, multiplayer oyunlardan sistem izleme (monitoring) panellerine kadar modern dünyanın hızı buradadır.

Bu başlığı; **Protokollerin Evrimi (Polling vs Push)**, **OSI Katmanındaki Yeri** ve **Ölçeklenme (Scaling) Problemleri** (C10k Sorunu) üzerinden, bir Mimar derinliğinde inceleyelim.

---

### 1. Evrim: "Orada mısın?"dan "Buradayım!"a

Gerçek zamanlı iletişim ihtiyacı yeni değil, ancak çözümlerimiz evrim geçirdi. Bir mimar olarak bu tekniklerin farkını ve maliyetini bilmelisin.

#### A. Regular Polling (Düzenli Yoklama) - "Eski Usül"

İstemci her 5 saniyede bir sunucuya sorar: "Yeni mesaj var mı?"

- **Sunucu:** "Yok." (5 saniye sonra) "Yok." (5 saniye sonra) "Var."
    
- **Maliyet:** Çok yüksek. Gereksiz ağ trafiği ve sunucu işlemci kullanımı. Gecikme (Latency) yüksektir (mesaj geldiğinde 4.9 saniye bekleyebilirsin).
    

#### B. Long Polling (Uzun Yoklama) - "Sabırlı Bekleyiş"

İstemci sorar: "Yeni mesaj var mı?"

- **Sunucu:** Cevap vermez! Bağlantıyı (Request) açık tutar ve bekletir (Pending).
    
- Ne zaman mesaj gelirse, sunucu cevabı döner ve bağlantıyı kapatır. İstemci cevabı alır almaz hemen yeni bir istek atar.
    
- **Maliyet:** Polling'den iyidir ama her mesajda yeni TCP bağlantısı kurma (Handshake) maliyeti vardır.
    

#### C. Server-Sent Events (SSE) - "Tek Yönlü Radyo"

HTTP standardıdır. İstemci bir kere bağlanır, sunucu o hat üzerinden sürekli veri akıtır (Stream).

- **Yön:** Tek yönlüdür (Server -> Client). İstemci aynı hattan cevap veremez.
    
- **Kullanım:** Borsa verileri, Haber akışı. (Chat için uygun değildir).
    

#### D. WebSockets - "Otoban"

İletişimin zirvesidir. HTTP olarak başlar (`Upgrade` header'ı ile), sonra **TCP** seviyesine iner.

- **Yön:** **Full-Duplex (Çift Yönlü).** Aynı anda hem sunucu hem istemci konuşabilir.
    
- **Maliyet:** Bağlantı bir kere kurulur, bir daha el sıkışma (Handshake) yapılmaz. Veri "Frame"ler halinde akar. En düşük gecikme (Low Latency) buradadır.
    

---

### 2. Mimari Zorluk: Stateful Connections (Durumlu Bağlantılar)

REST API'ler Stateless (Durumsuz) idi. Sunucu A çökse, Sunucu B isteği devralırdı.

Real-Time iletişim ise Stateful (Durumlu) olmak zorundadır.

- **Sorun:** Kullanıcı Ahmet, **Sunucu-1** ile WebSocket bağlantısı kurdu. Bu, Sunucu-1'in RAM'inde fiziksel bir TCP soketi demektir.
    
- **Risk:** Sunucu-1 çökerse, Ahmet'in bağlantısı kopar. Sunucu-2 bunu otomatik devralamaz çünkü soket fizikseldir. Ahmet'in tekrar bağlanması (Reconnect) gerekir.
    

---

### 3. Ölçeklenme Problemi: C10k ve Socket Exhaustion

Bir sunucu aynı anda kaç kişiyi canlı tutabilir?

Eskiden bu sınır 10.000 (C10k problemi) idi. Modern .NET (Kestrel) ile bu sayı milyonlara çıkabilir ama bir Liman (Port) sınırı vardır.

- Bir sunucuda ~65.000 port (Ephemeral Ports) vardır.
    
- Eğer her kullanıcı için bir port harcarsan, 65.000 kullanıcıda sistem kilitlenir.
    
- **Çözüm:** Modern Load Balancer'lar ve OS ayarlarıyla tek port üzerinden binlerce bağlantı yönetilir ama bellek (RAM) kullanımı her kullanıcı için artar (Her bağlantı ~2-4 KB bellek yer).
    

---

### 4. Teknoloji Seçimi: SignalR vs gRPC vs Raw WebSockets

.NET dünyasında 3 ana silahın var:

1. **ASP.NET Core SignalR:**
    
    - **Nedir:** Bir **Abstraction (Soyutlama)** kütüphanesidir.
        
    - **Gücü:** Alt tarafta en iyisini seçer. WebSocket varsa onu kullanır, yoksa SSE, o da yoksa Long Polling'e düşer (Fallback).
        
    - **Özellik:** "Hub" mantığı, Grup yönetimi ("Sadece Admin grubuna mesaj at"), Otomatik Reconnect.
        
    - **Kullanım:** Chat, Dashboard, Oyunlar. Standart seçimdir.
        
2. **gRPC Streaming:**
    
    - **Nedir:** HTTP/2 tabanlı binary akış.
        
    - **Gücü:** Performans.
        
    - **Kullanım:** Tarayıcı (Browser) desteği zayıftır. Genellikle **Backend-to-Backend** (Mikroservisler arası) canlı veri akışı için kullanılır.
        
3. **Raw WebSockets (`Microsoft.AspNetCore.WebSockets`):**
    
    - **Nedir:** Ham soket yönetimi. Byte array gönderir/alırsın.
        
    - **Gücü:** Maksimum kontrol ve minimum overhead.
        
    - **Zorluğu:** Protokolü (mesajın nerede başladığını bittiğini) sen yazarsın. Reconnect mantığını sen yazarsın.
        
    - **Kullanım:** Çok özel protokol gerektiren yüksek performanslı oyun sunucuları.
        

---

### 5. Mühendislik Harikası: The Backplane (Sırt Çantası)

Real-Time sistemlerin **Scaling (Büyüme)** konusundaki en kritik parçasıdır.

**Senaryo:**

- Siten çok popüler oldu, 2 sunucuya (Server A, Server B) geçtin.
    
- **Ahmet** -> Server A'ya bağlı.
    
- **Mehmet** -> Server B'ye bağlı.
    
- Ahmet, Mehmet'e mesaj attı. Mesaj Server A'ya geldi.
    
- **Sorun:** Server A, Mehmet'i tanımıyor (çünkü Mehmet Server B'nin belleğinde). Mesaj kaybolur.
    

Çözüm: Redis Backplane

Tüm sunucular birbirine Redis Pub/Sub üzerinden bağlanır.

1. Server A mesajı alır. "Mehmet bende yok" der.
    
2. Server A mesajı Redis'e atar: "Tüm sunuculara duyurulur! Mehmet diye biri varsa bu mesajı ona iletin."
    
3. Redis bu mesajı Server B'ye (ve diğerlerine) iletir.
    
4. Server B "Mehmet bende!" der ve mesajı Mehmet'in soketine yazar.
    

---

