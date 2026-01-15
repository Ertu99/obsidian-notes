### Analoji: Vale Anahtarı (Valet Key)

OAuth'u anlamanın en iyi yolu **Otel/Restoran Valesi** örneğidir.

Arabanı valeye teslim ederken ona "Ana Anahtarını" (Şifreni) vermezsin. Eğer ana anahtarı verirsen, torpidonu açabilir, bagajını karıştırabilir, hatta evinin anahtarı da oradaysa evine girebilir. Bunun yerine ona sadece kapıyı açan ve motoru çalıştıran **Vale Anahtarı**nı (Access Token) verirsin. Vale bu anahtarla arabayı park edebilir ama bagajı açamaz.

- **Sen:** Resource Owner (Kaynak Sahibi)
    
- **Vale:** Client (Senin Uygulaman)
    
- **Araba:** Resource Server (Korunan Veri)
    
- **Vale Anahtarı:** Access Token
    

---

### OAuth 2.0 Oyuncuları (Roles)

Bu terimler teknik dökümanlarda sürekli karşına çıkar:

1. **Resource Owner (Kaynak Sahibi):** Verinin asıl sahibi. (Kullanıcı, yani Sen).
    
2. **Client (İstemci):** Veriye erişmek isteyen uygulama. (Senin yazdığın Web Sitesi veya Mobil App).
    
3. **Authorization Server (Yetki Sunucusu):** Güvenliğin kalesi. Şifreyi soran ve Token'ı üreten taraf. (Google, Facebook veya senin kurduğun IdentityServer).
    
4. **Resource Server (Kaynak Sunucusu):** Verinin (API) bulunduğu yer. (Google Contacts API veya senin Protected API'ın).
    

---

### En Yaygın Senaryo: Authorization Code Flow

_(Kullanıcı "Google ile Giriş Yap" butonuna bastığında arka planda ne olur?)_

Bu akış, mülakatlarda adım adım sorulabilir.

1. **İstek:** Kullanıcı senin sitende "Google ile Bağlan"a tıklar.
    
2. **Yönlendirme:** Senin siten, kullanıcıyı Google'ın giriş sayfasına yönlendirir (`accounts.google.com`).
    
    - _Kritik Nokta:_ Kullanıcı şifresini senin sitene değil, **doğrudan Google'a** girer. Sen şifreyi asla görmezsin.
        
3. **İzin:** Google kullanıcıya sorar: _"Bu site senin rehberine erişmek istiyor. İzin veriyor musun?"_
    
4. **Code Dönüşü:** Kullanıcı "Evet" derse, Google kullanıcıyı tekrar senin sitene yönlendirir ve URL'in sonuna geçici bir **Code** ekler (`siteniz.com/callback?code=xyz123`).
    
5. **Takas (Exchange):** Senin Backend sunucun, bu "Code"u alır; Google'a gider ve _"Bak kullanıcı bana bu kodu verdi, karşılığında bana Token ver"_ der.
    
6. **Token:** Google kodu doğrular ve sana bir **Access Token** (genelde JWT) verir.
    
7. **Erişim:** Artık bu token ile Google API'larına gidip veriyi çekebilirsin.


### Backend Developer İçin Kritik İpuçları

#### 1. OAuth vs OpenID Connect (OIDC)

Bu ikisi hep karıştırılır.

- **OAuth 2.0:** Yetki içindir. "Bu uygulama benim fotoğraflarıma erişsin." (Anahtar).
    
- **OpenID Connect:** Kimlik doğrulama içindir. "Bu giriş yapan kişi Ahmet'tir." (Kimlik Kartı).
    
- Günümüzde "Google ile Giriş" dediğimiz şey aslında teknik olarak OAuth 2.0 üzerine inşa edilmiş **OIDC** katmanıdır.
    

#### 2. Refresh Token Hayat Kurtarır

Access Token'ların ömrü kısadır (örn: 1 saat). Kullanıcıyı her saat başı tekrar login ekranına göndermemek için **Refresh Token** kullanılır.

- Access Token süresi bitince, Backend sessizce Refresh Token'ı alıp Authorization Server'a gider ve "Bana yeni bir Access Token ver" der. Kullanıcı hiçbir kesinti hissetmez.
    

#### 3. Kendi OAuth Sunucunu Yazma!

Bir Junior hatası: "Ben kendi Identity sistemimi sıfırdan yazarım."

- **Yazma.** OAuth 2.0 ve OIDC çok karmaşık protokollerdir. En ufak bir açıkta sistemin hacklenir.
    
- Bunun yerine **IdentityServer (Duende)**, **OpenIddict** veya bulut çözümleri (**Auth0**, **Okta**, **Azure AD B2C**) kullan.
    

#### 4. Scope (Kapsam) Yönetimi

Kullanıcıdan sadece ihtiyacın olan yetkileri iste.

- Sadece email lazımsa, "Rehber Erişimi" veya "Fotoğraf Erişimi" isteme. Kullanıcılar gereksiz yetki isteyen uygulamalara güvenmez ve onayı iptal eder.