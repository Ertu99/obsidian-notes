**JWT (JSON Web Token)** → kullanıcı kimliğini dijital imzayla temsil eden bir string.

**Kullanım amacı:**

Backend, login olan kullanıcıya bir **token** üretir.

Sonraki her istek o token ile yapılır.

Backend bu token’ı doğrulayarak kullanıcının kim olduğunu bilir.

---

### 🧩 Token yapısı

JWT 3 parçadan oluşur:

```
xxxxx.yyyyy.zzzzz

```

|Bölüm|Anlam|Örnek|
|---|---|---|
|Header|Algoritma + tip bilgisi|`{ "alg": "HS256", "typ": "JWT" }`|
|Payload|Kullanıcı bilgileri (claims)|`{ "sub": "1", "email": "ertugrul@example.com", "role": "Admin" }`|
|Signature|Gizli anahtarla imzalanmış hash|HMACSHA256(header + payload + secretKey)|

---

### ⚙️ Çalışma akışı

1️⃣ Kullanıcı `POST /api/auth/login` → email + şifre gönderir

2️⃣ API kimliği doğrular

3️⃣ API bir JWT token üretip döner

4️⃣ Kullanıcı her istekle birlikte header’da token gönderir:

```
Authorization: Bearer <token>

```

5️⃣ API gelen token’ı doğrular → geçerliyse istek işlenir

## 9. JWT Arka Plan İşleyişi

Her istek geldiğinde:

1. **Middleware pipeline** içinde `UseAuthentication()` devreye girer.
2. `Authorization` header’ını kontrol eder.
3. Token çözülür → header + payload + signature ayrılır.
4. İmza (signature) `IssuerSigningKey` ile doğrulanır.
5. Eğer geçerliyse `HttpContext.User` oluşturulur → `[Authorize]` attribute bunu kullanır.
6. Token süresi dolmuşsa → 401 Unauthorized döner.

---

## 🔹 10. Mülakat notları

| Soru                                          | Cevap Özeti                                                                       |
| --------------------------------------------- | --------------------------------------------------------------------------------- |
| JWT nedir?                                    | Kullanıcı kimliğini JSON formatında imzalı olarak taşıyan token.                  |
| Token nereye konur?                           | Header → `Authorization: Bearer <token>`.                                         |
| ValidateIssuer/ValidateAudience ne işe yarar? | Token’ın doğru uygulamadan geldiğini doğrular.                                    |
| Token imzası nasıl çalışır?                   | Header+Payload gizli anahtarla hashlenir, doğrulama sırasında aynı hash üretilir. |
| `[Authorize]` ne yapar?                       | Context’teki kimliği kontrol eder, yoksa 401 döner.                               |