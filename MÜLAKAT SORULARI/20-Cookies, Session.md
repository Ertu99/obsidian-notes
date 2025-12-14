## 1️⃣ Cookies (Çerezler) Nedir?

**Tanım:**

> Cookie, kullanıcının tarayıcısında (client-side) saklanan küçük veri parçacıklarıdır.

Genellikle kullanıcıyı tanımak veya oturum bilgilerini hatırlamak için kullanılır.

---

### 💡 Örnek:

Sen bir siteye giriş yaptın →

Sunucu tarayıcıya şöyle bir **cookie** gönderir:

```
session_id=abc123; expires=Fri, 07-Nov-2025 12:00:00 GMT;

```

Tarayıcı bunu saklar ve **her istekte sunucuya otomatik gönderir.**

### 📦 Örnek kullanım alanları:

- “Beni hatırla” seçeneği
- Oturum kimliği (`session_id`)
- Tema, dil, kullanıcı tercihleri

---

### ⚙️ Özellikleri:

- Tarayıcıda saklanır (client tarafı).
- Maksimum boyutu ~4 KB civarındadır.
- Süreli olabilir (`expires` veya `max-age`).
- Kullanıcı tarafından **silinebilir** veya **değiştirilebilir.**

## 🧩 2️⃣ Session (Oturum) Nedir?

**Tanım:**

> Session, kullanıcının verilerinin sunucu tarafında (server-side) geçici olarak tutulduğu alandır.

Her kullanıcıya özgü bir **Session ID** oluşturulur, bu ID cookie içinde saklanır.

---

### 💡 Örnek:

Bir kullanıcı giriş yaptı →

Sunucu `Session["UserId"] = 5` olarak kaydeder.

Tarayıcıya küçük bir cookie gönderilir:

```
ASP.NET_SessionId=xyz987

```

Bu ID ile sunucu o kullanıcının bilgilerine ulaşabilir.

---

### 📦 Örnek kullanım alanları:

- Kullanıcı giriş bilgileri
- Sepet verileri (e-ticaret)
- Kullanıcıya özel geçici ayarlar

---

### ⚙️ Özellikleri:

- Sunucuda saklanır (client değil).
- Tarayıcı kapatılınca veya süre dolunca silinir.
- Cookie’den daha **güvenlidir** çünkü kullanıcı tarafından doğrudan değiştirilemez.
- Fazla veri tutmak sunucu yükünü artırabilir.