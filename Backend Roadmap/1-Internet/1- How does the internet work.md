
İlk konumuz olan **"How does the Internet work?" (İnternet Nasıl Çalışır?)** ile başlıyoruz. Bu, bir Backend Developer'ın üzerine inşa edeceği temel zemindir. Eğer zemini sağlam atarsak, ileride göreceğimiz HTTP, DNS, Hosting gibi konular çok daha rahat anlaşılır.

> "İnternet, dünya üzerindeki milyarlarca cihazın birbirine bağlanmasını sağlayan **küresel bir ağlar ağıdır (Network of Networks)**.
> 
> Temel çalışma prensibi, **Client-Server (İstemci-Sunucu)** modeline dayanır. Ben bir kullanıcı olarak tarayıcımdan bir istek (Request) gönderdiğimde, bu istek önce **ISP** (İnternet Servis Sağlayıcım) üzerinden ağa çıkar. Gideceği adresi bulmak için **DNS** (Domain Name System) sunucularını kullanır; yani alan adını (https://www.google.com/search?q=google.com) makine diline, yani IP adresine çevirir.
> 
> Hedef sunucunun adresi bulunduğunda, verilerim küçük **paketlere** bölünür ve **TCP/IP** protokolleri sayesinde, yönlendiriciler (router'lar) üzerinden en hızlı yoldan hedef sunucuya ulaşır. Sunucu isteği işler ve cevabı (Response) bana geri gönderir. Backend developer olarak bizim işimiz, o sunucuya ulaşan isteği karşılayıp doğru cevabı üretmektir."

**Neden Kullanılır?** Bilgi paylaşımı, iletişim ve dağıtık sistemlerin (distributed systems) birbiriyle konuşabilmesi için kullanılır. Merkezi olmayan (decentralized) yapısı sayesinde, bir hat kopsa bile veri başka yollardan hedefe ulaşabilir.

---

### Bölüm 2: Derinlemesine Teknik Anlatım (Deep Dive)

Şimdi kaputu açalım. Bir .NET Junior Developer olarak, yazdığın API'nin dünyaya nasıl açıldığını anlaman için burayı atomlarına ayıracağız. Konuyu kavramsal parçalara ve bir senaryoya böleceğim.

#### 1. Fiziksel Altyapı: İnternet "Bulut" Değildir, Kablodur!

Çoğu insan interneti havada uçuşan bir şey sanır ama internet %99 oranında **fiziksel kablolardan** oluşur.

- **Omurga (Backbone):** Okyanusların altında yatan devasa fiber optik kablolar kıtaları birbirine bağlar.
    
- **Last Mile (Son Kilometre):** Evine gelen bakır kablo, fiber veya baz istasyonundan telefonuna gelen sinyal.
    
- **Router (Yönlendirici):** İnternetin trafik polisleridir. Veri paketlerinin A noktasından B noktasına giderken hangi yoldan gitmesi gerektiğine karar verirler.
    

#### 2. İnternetin Dili: Protokoller

İnternet üzerindeki cihazların anlaşabilmesi için ortak bir dile ihtiyacı vardır. Buna **Protokol** denir.

- **TCP/IP (Transmission Control Protocol / Internet Protocol):** İnternetin anayasasıdır.
    
    - **IP:** Adresleme sistemidir (Paketin nereye gideceğini belirler).
        
    - **TCP:** Taşıma işidir (Paketin eksiksiz ve sırasıyla gidip gitmediğini kontrol eder).
        
- **HTTP/HTTPS:** Web sitelerinin görüntülenmesi için kullanılan uygulama katmanı protokolüdür (İleride detaylı göreceğiz).
    

#### 3. Adresleme: IP Adresleri ve DNS

İnternete bağlı **her** cihazın (bilgisayar, sunucu, akıllı süpürge) benzersiz bir kimlik numarası vardır. Buna **IP Adresi** denir (Örn: `192.168.1.1` veya `142.250.185.78`).

- **Sorun:** İnsanlar sayıları ezberleyemez. "Bugün `142.250.185.78`'e girip bir şeyler arayayım" demeyiz, "https://www.google.com/search?q=google.com" deriz.
    
- **Çözüm (DNS):** İnternetin telefon rehberidir. Sen tarayıcıya "https://www.google.com/search?q=google.com" yazdığında, tarayıcın DNS sunucusuna gider ve "Bu ismin IP adresi nedir?" diye sorar. DNS de "Bunun adresi X'tir" der.
    

#### 4. Paket Anahtarlama (Packet Switching) - _Mülakat Kritik Noktası_

Bu, internetin verimliliğinin sırrıdır. Diyelim ki sunucuya 10 MB'lık bir resim yüklüyorsun. Bu resim tek bir devasa blok halinde gitmez.

- Veri binlerce küçük **Pakete (Packet)** bölünür.
    
- Her paketin üzerine "Benim sıram 1, hedefim X", "Benim sıram 2, hedefim X" gibi etiketler (Header) yapıştırılır.
    
- **Önemli:** Paketler aynı yoldan gitmek zorunda değildir! 1. paket Almanya üzerinden, 2. paket İtalya üzerinden gidebilir.
    
- Hedefe vardıklarında **TCP** bu paketleri sıraya dizer ve birleştirir. Eğer bir paket kaybolduysa, TCP "3 numaralı paket gelmedi, tekrar gönder" der.
    

---

### Bölüm 3: Bir İsteğin Yolculuğu (Step-by-Step Life Cycle)

_Mülakatlarda "Tarayıcıya https://www.google.com/search?q=google.com yazdığında ve enter'a bastığında arka planda neler olur?" sorusunun en detaylı cevabı budur. Bunu bir Backend'ci gözüyle anlatıyorum:_

1. **Tarayıcı (Browser):**
    
    - Kullanıcı `roadmap.sh` yazar.
        
    - Tarayıcı önce kendi önbelleğine (cache) bakar: "Ben bu sitenin IP adresini biliyor muyum?"
        
    - Bilmiyorsa İşletim Sistemine sorar.
        
2. **DNS Çözümleme (Lookup):**
    
    - İşletim sistemi de bilmiyorsa, isteği **ISP**'nin (Superonline, TurkTelekom vb.) DNS sunucusuna (Resolver) gönderir.
        
    - DNS hiyerarşisi çalışır: Root Server -> TLD Server (.sh) -> Authoritative Nameserver (roadmap.sh'ın gerçek adresi).
        
    - Sonuç olarak IP adresi bulunur: Örn: `104.21.23.45`.
        
3. **TCP Handshake (El Sıkışma):**
    
    - Tarayıcı artık hedef IP'yi biliyor. Sunucuyla (Backend) iletişim kurmak için bir oturum açar.
        
    - Buna **3-Way Handshake** denir:
        
        1. Client: "Selam, konuşabilir miyiz?" (SYN)
            
        2. Server: "Selam, evet konuşabiliriz." (SYN-ACK)
            
        3. Client: "Tamam, gönderiyorum." (ACK)
            
4. **İstek Gönderimi (HTTP Request):**
    
    - Bağlantı kuruldu. Tarayıcı sunucuya şöyle bir mektup (Request) gönderir:
        
        - _GET /backend-roadmap HTTP/1.1_
            
        - _Host: roadmap.sh_
            
    - Bu mektup paketlere bölünür, kablolardan geçer, router'lardan atlar ve sunucuya ulaşır.
        
5. **Sunucu Tarafı (Backend - Bizim Alanımız):**
    
    - Bu IP adresinde çalışan bir Web Sunucusu (Örn: Nginx, IIS, Apache) isteği karşılar.
        
    - Eğer bu bir .NET Core uygulamasıysa, istek **Kestrel** (dahili sunucu) üzerinden uygulamanın içine (Pipeline) girer.
        
    - Senin yazdığın Controller veya Minimal API kodu çalışır. Veritabanına gidilir, veri çekilir.
        
6. **Cevap Dönüşü (HTTP Response):**
    
    - Senin kodun bir HTML sayfası veya JSON verisi üretir.
        
    - Sunucu bunu paketleyip aynı yoldan geriye, Client'a gönderir.
        
7. **Render (Görüntüleme):**
    
    - Tarayıcı paketleri birleştirir. HTML/CSS/JS kodlarını yorumlar ve kullanıcıya sayfayı çizer.
        

---

### Özet: Backend Developer İçin Neden Önemli?

Bir .NET developer olarak sen, bu zincirin **5. ve 6. adımlarında** yaşıyorsun. Ama şunları bilmek zorundasın:

- **DNS** sorunları yüzünden sitene erişilemeyebilir.
    
- **Paket kaybı (Latency)** yüzünden API'lerin yavaş çalışabilir.
    
- **TCP/IP** mantığını bilmezsen, neden SignalR (WebSockets) kullandığımızı anlayamazsın.
    

Bu konu, internetin temel omurgasıydı. Buradaki kavramların çoğu (DNS, HTTP, Hosting) kendi başlıklarında tekrar detaylanacak ama "Büyük Resim" (Big Picture) budur.
