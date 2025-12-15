DNS (Domain Name System), internetin telefon rehberidir. İnsanların kolayca hatırlayabildiği alan adlarını (örneğin `roadmap.sh`), bilgisayarların birbirini bulmak için kullandığı IP adreslerine (örneğin `104.21.23.45`) çeviren sistemdir.

Neden kullanılır? Çünkü bilgisayarlar sayılarla (IP) iletişim kurar, insanlar ise kelimelerle (Domain) hatırlar. Eğer DNS olmasaydı, girmek istediğin her web sitesinin IP adresini bir deftere not edip ezberlemek zorunda kalırdın.

---

### Deep Dive: DNS Nasıl Çalışır? (Sorgu Zinciri)

Mülakatlarda "Tarayıcıya adresi yazınca ne olur?" sorusunun en teknik kısmı burasıdır. DNS, tek bir merkezden yönetilmez; **hiyerarşik** ve **dağıtık** bir yapısı vardır.

#### 1. DNS Hiyerarşisi (Kim kime soruyor?)

Sistem tepeden aşağıya doğru iner:

- **Root Nameservers (Kök Sunucular):** En tepedeki patronlardır. Dünyada 13 ana kök sunucu grubu vardır (A'dan M'ye harflendirilir). Bunlar "Ben `roadmap.sh`'ın adresini bilmem ama `.sh` ile bitenlere kimin baktığını biliyorum" derler.
    
- **TLD Nameservers (Top-Level Domain):** Uzantılardan sorumlu sunuculardır (.com, .net, .sh sunucuları). "Ben `roadmap.sh`'ın tam IP'sini bilmem ama bu domaini kimin yönettiğini (Authoritative Server) biliyorum" derler.
    
- **Authoritative Nameservers (Yetkili Sunucular):** Domaini satın aldığın veya yönlendirdiğin firmanın (Cloudflare, AWS, GoDaddy) sunucularıdır. "Evet, `roadmap.sh` bende kayıtlı ve IP adresi şudur" diyen nihai merci burasıdır.
    

#### 2. DNS Çözümleme Süreci (Resolution Steps)

Sen tarayıcıya `www.google.com` yazdığında arka planda saniyeler içinde şu konuşma gerçekleşir:

1. **Local Cache (Yerel Önbellek):** Bilgisayarın önce kendine bakar: "Ben buna daha önce girdim mi?" Eğer girdiyse IP'yi hafızasından verir. (İşlem biter).
    
2. **DNS Resolver (ISP):** Bilgisayarında yoksa, isteği internet servis sağlayıcına (Superonline, Turknet vb.) gönderir. Onların sunucusu (Resolver) devreye girer.
    
3. **Root Server'a Soru:** Resolver, Kök sunucuya gider: "Bana `google.com`'u bul."
    
    - _Cevap:_ "Ben bilmem, `.com` TLD sunucusuna git."
        
4. **TLD Server'a Soru:** Resolver, `.com` sunucusuna gider.
    
    - _Cevap:_ "Adresi tam bilmem ama bu domainin kayıtları `ns1.google.com` sunucusunda."
        
5. **Authoritative Server'a Soru:** Resolver, yetkili sunucuya gider.
    
    - _Cevap:_ "Buldum! IP adresi `142.250.185.78`."
        
6. **Sonuç:** Resolver bu IP'yi alır, hem sana (bilgisayarına) verir hem de bir süreliğine kendi hafızasına (Cache) yazar ki bir daha uğraşmasın.
    

#### 3. TTL (Time To Live - Yaşam Süresi)

DNS kayıtlarında **TTL** diye bir değer vardır. Bu, bir kaydın önbellekte (cache) ne kadar süre tutulacağını belirtir.

- TTL 3600 (saniye) ise; DNS sunucuları bu bilgiyi 1 saat hafızada tutar.
    
- Sen IP adresini değiştirsen bile, dünyadaki diğer kullanıcılar 1 saat boyunca eski IP'ye gitmeye devam edebilir. Buna **DNS Propagation (Yayılma)** denir.
    

---

### Backend Developer İçin Neden Önemli?

1. **Veritabanı Bağlantıları:** Projende Connection String yazarken genellikle IP yerine sunucu ismini (`db.prod.local` gibi) kullanırsın. Burada da içeride çalışan bir DNS mekanizması vardır. Eğer DNS sunucun yavaşsa, uygulamanın veritabanına bağlanması gecikir.
    
2. **Latency (Gecikme):** Kullanıcın API'ne istek attığında, DNS çözümlemesi 200ms sürüyorsa, senin kodun ne kadar hızlı olursa olsun kullanıcı o gecikmeyi yaşar.
    
3. **HttpClient Kullanımı:** .NET Core içinde `HttpClient` ile başka bir API'ye istek atarken, `HttpClient` varsayılan olarak DNS değişikliklerini hemen algılamayabilir (eski versiyonlarda). Bu yüzden `IHttpClientFactory` kullanmak best practice'tir; çünkü DNS yenilemelerini doğru yönetir.
    


