
---

### 1. MVC Nedir? (Felsefesi: Separation of Concerns)

Eskiden (ASP.NET Web Forms zamanlarında), bir web sayfasının kodu "spagetti" gibiydi. Veritabanı bağlantısı, HTML kodları ve C# mantığı aynı dosyanın içindeydi (`Page_Load` eventleri). Birini değiştirsen diğeri bozulurdu.

MVC, yazılım mühendisliğindeki **"Separation of Concerns" (İlgi Alanlarının Ayrımı)** prensibini uygular.

- **Felsefe:** "Herkes kendi işini yapsın."
    
- **Amaç:** Yönetilebilirlik, Test Edilebilirlik ve Takım Çalışması. (Frontendci View ile uğraşırken, Backendci Controller ile uğraşabilir).
    

![MVC architectural pattern diagram resmi](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcSwvQvasxKragYnplEMXdWbXM7BKaaN20syQc0W_iRYa5Lwej3-KiI1csiRG_PCKvEoHfcNn7DjWMX-lwXVAWVgqjRcTseaGz8XD0vra-5n-hzOggE)

Shutterstock

---

### 2. Anatomisi: 3 Temel Katman

Verilen metinde tanımlar var, ama biz bunların **teknik sorumluluklarına** ve **yapmamaları gereken** hatalara odaklanalım.

#### A. Controller (Beyin / Trafik Polisi)

İstekleri karşılayan ilk noktadır.

- **Görevi:** Gelen isteği (`Request`) analiz eder, hangi işin yapılacağına karar verir, Model'den veriyi ister ve sonucu View'a gönderir.
    
- **Mühendislik Kuralı (Skinny Controller):** Controller **asla** iş mantığı (Business Logic) içermemelidir. "Kullanıcı 18 yaşından büyük mü?", "Stokta ürün var mı?" gibi kontrolleri Controller yapmaz. O sadece Service katmanına "Bunu kontrol et" der.
    
- **Analoji:** Restorandaki **Garson**. Siparişi alır, mutfağa (Model/Service) iletir, yemeği (View) getirir. Yemeği kendisi pişirmez.
    

#### B. Model (Veri ve Mantık)

Uygulamanın kalbidir.

- **Görevi:** Veriyi temsil eder ve iş kurallarını (Business Rules) taşır.
    
- **İki Türü Vardır (Önemli Ayrım):**
    
    1. **Domain Model:** Veritabanındaki tablonun birebir karşılığıdır (`User`, `Product`).
        
    2. **View Model:** Sadece o ekranda gösterilecek veridir. (Örn: `UserLoginViewModel` içinde sadece Email ve Şifre olur, Doğum Tarihi olmaz).
        
- **Analoji:** Restorandaki **Mutfak ve Malzemeler**.
    

#### C. View (Yüz / Arayüz)

Kullanıcının gördüğü kısımdır.

- **Görevi:** Controller'dan gelen veriyi alır ve HTML'e dönüştürür.
    
- **Teknolojisi:** ASP.NET Core'da **Razor View Engine** (.cshtml) kullanılır. HTML içine C# yazmamızı sağlar.
    
- **Kural (Dumb View):** View "akılsız" olmalıdır. İçinde karmaşık C# mantığı, veritabanı sorgusu asla olmamalıdır. Sadece `foreach` döngüsü veya `if` ile gösterme/gizleme yapmalıdır.
    
- **Analoji:** Restorandaki **Tabak Sunumu**.
    

---

### 3. İstek Yaşam Döngüsü (The Lifecycle)

Bir kullanıcı tarayıcıya `www.site.com/Urunler/Detay/5` yazdığında arka planda ne olur? Bu akışı bilmek zorundasın.

1. **Routing (Yönlendirme):** İstek sunucuya gelir. ASP.NET Core Route motoru bakar:
    
    - `Urunler` -> Controller Adı (`UrunlerController`)
        
    - `Detay` -> Action (Metot) Adı (`Detay()`)
        
    - `5` -> Parametre (`id`)
        
2. **Controller Action:** `UrunlerController` sınıfındaki `Detay(int id)` metodu çalışır.
    
3. **Service/Model Call:** Controller, servise gidip "Bana ID'si 5 olan ürünü getir" der.
    
4. **Result:** Servis veriyi (Model) döndürür.
    
5. **View Selection:** Controller, bu veriyi `View(model)` diyerek sunum katmanına atar.
    
6. **Rendering:** Razor motoru HTML'i oluşturur ve kullanıcıya gönderir (Response).
    

---

### 4. Modern Dünyada MVC: API vs Web App

Metinde "Specifically web applications" dese de, bugün MVC iki şekilde kullanılır:

1. **MVC Web App:** Geleneksel yöntem. HTML'i sunucu üretir (Server Side Rendering). SEO için iyidir.
    
2. **Web API (Controller + Model, No View):** Modern yöntem.
    
    - **Controller:** İsteği alır.
        
    - **Model:** Veriyi işler.
        
    - **View YOK:** Onun yerine saf **JSON** verisi döner.
        
    - Bu JSON'ı kim karşılar? React, Vue, Angular veya Mobil Uygulama.
        

Yani API yazarken de aslında MVC desenini (V'si eksik olsa da) kullanıyorsun.

---

