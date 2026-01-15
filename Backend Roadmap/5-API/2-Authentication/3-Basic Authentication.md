JWT ve OAuth gibi modern ve havalı yöntemleri öğrendik. Şimdi ise tarihin tozlu sayfalarından gelen ama hala ölmemiş olan dedelerine, **Basic Authentication**'a bakacağız.

Genellikle "Eski moda" olarak görülse de, Backend geliştirici olarak entegrasyon yaparken (özellikle eski banka sistemleri veya basit iç servislerde) karşına kesinlikle çıkacaktır.

---

### Basic Auth Nedir?

Basic Authentication, bir istemcinin (Client) kimliğini doğrulamak için **Kullanıcı Adı** ve **Şifre** bilgisini her istekte (Request) sunucuya göndermesidir.

JWT gibi karmaşık imzalama süreçleri, OAuth gibi yönlendirmeler yoktur. Dümdüz, "Ben Ahmet, şifrem 1234, beni içeri al" demektir.

### Nasıl Çalışır? (Teknik Detay)

Sistem aslında çok basit bir string manipülasyonuna dayanır:

1. **Birleştirme:** Kullanıcı adı ve şifre araya iki nokta üst üste (`:`) konularak birleştirilir.
    
    - `username:password` -> `admin:123456`
        
2. **Kodlama (Encoding):** Bu birleşik metin **Base64** formatına çevrilir.
    
    - `admin:123456` -> `YWRtaW46MTIzNDU2`
        
3. **Gönderim:** HTTP Header'ına eklenir.
    
    - `Authorization: Basic YWRtaW46MTIzNDU2`
        

---

### Büyük Güvenlik Tuzağı: Base64 Şifreleme Değildir!

Bir Junior Developer'ın düşeceği en büyük hata şudur: _"Abi şifreyi Base64'e çevirdim, karmaşık bir yazı oldu, artık güvende."_

**HAYIR!** Base64 bir şifreleme (Encryption) yöntemi değil, bir kodlama (Encoding) yöntemidir.

- **Encryption:** Anahtar olmadan geri döndürülemez (AES, RSA).
    
- **Encoding:** Herkes geri döndürebilir (Base64, Hex).
    

Google'a "Base64 Decode" yazan herhangi biri, senin `YWRtaW46MTIzNDU2` string'ini saniyesinde `admin:123456` olarak okuyabilir.

🛑 **Altın Kural:** Basic Authentication kullanıyorsan, sunucun **MUTLAKA HTTPS (SSL)** olmak zorundadır. HTTPS, trafiği şifreleyerek bu Base64 string'in yolda (Man-in-the-Middle) okunmasını engeller.

### Ne Zaman Kullanmalı? Ne Zaman Kullanmamalı?

Burası bir Backend Mühendisi olarak vereceğin mimari karardır.

**✅ Kullan:**

- **İç Servisler (Internal Microservices):** Dışarıya kapalı, sadece kendi sunucularının birbiriyle konuştuğu, VPN arkasındaki servislerde hızlı ve kolaydır.
    
- **Hızlı Prototip:** POC (Proof of Concept) yaparken JWT kurmakla uğraşmak istemediğinde.
    
- **Basit Scriptler:** Bir Python scripti yazıp API'ye veri atacağın zaman (Token al, süresini kontrol et vs. uğraşmamak için).
    

**❌ KULLANMA:**

- **Mobil Uygulamalar:** Şifreyi telefonda saklaman gerekir. Telefon çalınırsa şifre de gider. (JWT/OAuth kullan).
    
- **Web Frontend (SPA):** Şifreyi tarayıcıda saklamak risklidir. Ayrıca her istekte şifre göndermek performansı düşürmez ama güvenlik riskini artırır.
    
- **Logout Gereken Yerler:** Basic Auth'da "Çıkış Yap" (Logout) diye bir şey yoktur. Tarayıcıyı kapatana kadar tarayıcı o şifreyi hatırlar ve göndermeye devam eder.
    

---

**Özet:** Basic Auth, "Kullanıcı Adı + Şifre" ikilisinin Base64 ile paketlenip gönderilmesidir. HTTPS olmadan kullanmak intihardır.