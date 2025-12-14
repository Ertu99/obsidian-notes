
Şimdi Testing dünyasının son kalesi, piramidin zirvesi ve en pahalı (ama en gerçekçi) testi olan **E2E (End-to-End / Uçtan Uca) Testing** konusuna ve bu işin modern kralı **Playwright**'a geliyoruz.

Selenium yıllarca bu işin standardıydı ama yavaştı, kırılgandı (flaky) ve kurulumu zordu. Microsoft, "Bu iş böyle olmaz" diyerek Playwright'ı çıkardı.

Playwright sadece bir "Tıklama Aracı" değildir. O, modern web'i (React, Angular, Blazor) anlayan, ağ trafiğini dinleyen (Network Interception) ve paralel çalışabilen bir Browser Automation canavarıdır.

Bu konuyu; **Architecture (Mimari)**, **Auto-Waiting (Otomatik Bekleme)**, **Tracing (Hata İzleme)** ve **Parallelism** üzerinden, tek seferde ve eksiksiz inceleyelim.

---

### 1. Felsefe: Neden Selenium Değil?

Selenium, HTTP tabanlı bir protokol (WebDriver) kullanır. Her komut için tarayıcıya HTTP isteği atar.

- **Playwright:** WebSocket üzerinden tarayıcıyla (CDP - Chrome DevTools Protocol) **doğrudan** konuşur.
    

|**Özellik**|**Selenium**|**Playwright**|
|---|---|---|
|**Hız**|Yavaş (HTTP Roundtrips)|Çok Hızlı (WebSocket)|
|**Bekleme**|Manuel (`Thread.Sleep` veya `WaitUntil`)|Otomatik (Auto-Wait)|
|**Network**|Zor (Proxy gerekir)|Yerleşik (API Mocking)|
|**Browser**|Driver kurulumu gerekir|Binaries içinde gelir|
|**Headless**|Destekler|Varsayılan (Çok hızlı)|

---

### 2. Mimari: Browser, Context ve Page

Playwright'ın nesne hiyerarşisi, testlerin izolasyonu ve hızı için kritik öneme sahiptir.

1. **Browser:** Tarayıcı motorudur (Chromium, Firefox, WebKit). Çok pahalıdır, test başında 1 kere başlatılır.
    
2. **BrowserContext (İzolasyonun Sırrı):** "Gizli Sekme" (Incognito) gibidir.
    
    - Her test için **yeni bir Context** açılır.
        
    - Çerezler, LocalStorage ve Cache **Context içinde izoledir.**
        
    - Test A giriş yaparsa, Test B bunu görmez. (Selenium'da bu zordu).
        
    - Oluşturması milisaniyeler sürer.
        
3. **Page:** Sekmenin kendisidir.
    

C#

```cs
// Test Başlangıcı
var browser = await playwright.Chromium.LaunchAsync();
var context = await browser.NewContextAsync(); // Yeni Profil (Hızlı)
var page = await context.NewPageAsync();       // Yeni Sekme

await page.GotoAsync("https://mysite.com");
```

---

### 3. Mühendislik Harikası: Auto-Waiting (Otomatik Bekleme)

E2E testlerin en büyük düşmanı **"Flaky Test"**tir (Bazen geçen, bazen kalan test).

- _Sebep:_ Sen "Tıkla" dersin ama buton henüz JavaScript tarafından render edilmemiştir veya animasyon devam ediyordur.
    
- _Eski Çözüm:_ `Thread.Sleep(5000)` (İğrençtir).
    

Playwright Çözümü:

Sen await page.ClickAsync("#btn-submit") dediğinde Playwright şunları otomatik kontrol eder:

1. Element DOM'da var mı?
    
2. Görünür mü (Visible)?
    
3. Animasyonu bitti mi (Stable)?
    
4. Önünde başka bir element (Overlay) var mı?
    
5. Enabled mı?
    

Eğer değilse, varsayılan olarak **30 saniye** boyunca (Retry) dener. Senin `Wait` yazmana gerek kalmaz.

---

### 4. Code Gen (Kod Üretici)

Test yazmak amelelik olmamalı. Playwright, senin tarayıcıdaki hareketlerini kaydedip C# koduna çeviren bir araca sahiptir.

- **Komut:** `playwright codegen wikipedia.org`
    
- **İşlem:** Sen sitede gezer, arama yapar, tıklarsın.
    
- **Sonuç:** Yan ekranda C# kodun (xUnit uyumlu) otomatik yazılır.
    
- **Kullanım:** Temel iskeleti oluşturmak için harikadır, sonra kodu temizlersin.
    

---

### 5. Network Interception (API Mocking)

E2E test yaparken Backend'in hazır olmayabilir veya 3. parti API (Google Maps) çok yavaş olabilir. Playwright ile **ağ trafiğini kesip** sahte cevap dönebilirsin.

C#

```cs
// "api/users" isteği gelirse sunucuya gitme, bu JSON'ı dön!
await page.RouteAsync("**/api/users", route => 
{
    route.FulfillAsync(new RouteFulfillOptions 
    {
        Status = 200,
        Body = "[{ 'name': 'Sahte Ahmet' }]",
        ContentType = "application/json"
    });
});

await page.GotoAsync("/users"); // Sayfa "Sahte Ahmet"i gösterir.
```

Böylece Frontend'i Backend'den bağımsız (Isolated) test edebilirsin.

---

### 6. Hata Ayıklama: Trace Viewer

Testin CI sunucusunda patladı. Neden?

Loglara bakmak yetmez. Playwright Trace Viewer ile sana zaman yolculuğu yaptırır.

- Testin her adımı için (Before/After) ekran görüntüsü alır.
    
- Her adımda ağ trafiğini (Network), konsol loglarını ve DOM yapısını kaydeder.
    
- Sen bir "Timeline" üzerinde ileri geri gidip hatanın olduğu anı film gibi izlersin.
    

---

### 7. Paralel Çalıştırma (Parallelism)

xUnit ile entegre olduğunda, Playwright testleri (Context yapısı sayesinde) %100 izole olarak paralel çalıştırabilir.

- 4 Çekirdekli bir işlemcide, aynı anda 4 farklı "Gizli Sekme" açılır ve 4 test aynı anda koşar. Test süresi 4 kat kısalır.
    

---

**🧒 6 Yaşındaki Çocuğa (Uzaktan Kumandalı Araba vs Hayalet Sürücü):** "Eskiden web sitelerini test etmek için **Selenium** adında bir robot kullanırdık. Bu robotun elinde bir kumanda vardı. Düğmeye basardı, sinyal uzaktaki arabaya (tarayıcıya) giderdi, araba hareket ederdi. Bu sinyal bazen gecikirdi, araba duvara çarpardı. Çok yavaştı. **Playwright** ise arabanın içine giren bir **Hayalet Sürücü** gibidir. Kumandaya ihtiyacı yoktur, zaten arabanın beyninin içindedir (**WebSocket/CDP**). Gaza bastığı an araba uçar. Ayrıca bu hayalet çok sabırlıdır. Eğer trafik lambası kırmızıysa (**Auto-Waiting**), eski robot gibi gaza basıp kaza yapmaz; yeşil yanana kadar bekler ve öyle geçer. Test yaparken de her seferinde yeni bir araba satın almaz. Arabanın koltuk kılıflarını değiştirir (**Browser Context**), sanki yeniymiş gibi kullanır. Çok daha hızlı ve ucuzdur."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** "Test Piramidi'nin zirvesinde yer alan E2E testleri, kullanıcı deneyimini doğrulayan en gerçekçi ama aynı zamanda en maliyetli ve kırılgan (Flaky) katmandır. Endüstri standardı uzun yıllar Selenium olsa da, modern Single Page Application (SPA) mimarilerinde yaşanan senkronizasyon sorunları ve hantallık nedeniyle mimari tercih **Playwright** yönüne kaymaktadır. Bu geçişin temelindeki mühendislik sebepleri şunlardır:

- **Protocol Efficiency:** Selenium'un HTTP tabanlı JSON Wire protokolü yerine, Playwright'ın doğrudan tarayıcı motoruyla (CDP) WebSocket üzerinden haberleşmesi, test hızını ve kararlılığını dramatik şekilde artırır.
    
- **Auto-Waiting Mechanism:** E2E testlerin en büyük hastalığı olan 'Flaky Test' (istikrarsızlık) sorunu, Playwright'ın DOM elemanlarının hazır olmasını (Visible, Stable, Enabled) otomatik beklemesi sayesinde kod kirliliği yaratmadan (`Thread.Sleep` olmadan) çözülür.
    
- **Isolation Strategy:** Her test için ağır bir tarayıcı süreci başlatmak yerine, **Browser Context** (Incognito benzeri yapı) kullanılarak milisaniyeler içinde izole ortamlar yaratılır. Bu da testlerin paralel (Parallel Execution) ve birbirini etkilemeden koşulmasını sağlar.
    
- **Network Mocking:** Backend servislerinin hazır olmadığı veya yavaş olduğu durumlarda, ağ trafiğini (Network Interception) manipüle ederek Frontend testlerinin bağımsız (Hermetic Testing) yapılabilmesine olanak tanır."