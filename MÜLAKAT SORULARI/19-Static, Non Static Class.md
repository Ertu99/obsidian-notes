## 🧩 1️⃣ Normal (Non-Static) Class

➡️ **Bir nesne oluşturularak** kullanılabilir.

Yani “new” anahtar kelimesiyle bir **örneği (instance)** oluşturulur.

---

### 🔹 Örnek:

```csharp
class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

```

### 🔹 Kullanım:

```csharp
Calculator calc = new Calculator();
int result = calc.Add(3, 5);
Console.WriteLine(result); // 8

```

🧠 Açıklama:

- Her `new Calculator()` çağrısında **yeni bir nesne** oluşur.
- Her nesnenin kendi değişkenleri (state) olabilir.
- **Nesneye özel veriler** tutmak mümkündür.

---

## 🧩 2️⃣ Static Class

➡️ **Nesne oluşturulamaz.**

Tüm üyeleri (metot, property, field) **class düzeyinde (global)** çalışır.

---

### 🔹 Örnek:

```csharp
static class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}

```

### 🔹 Kullanım:

```csharp
int result = MathHelper.Add(10, 20); // Nesne oluşturulmadan çağrılır
Console.WriteLine(result); // 30

```

🧠 Açıklama:

- `new MathHelper()` yazamazsın → ❌ derleme hatası.
- `static` üyeler **tüm uygulamada ortaktır (tek kopya)**.
- Bellekte sadece **bir kez** yer kaplar.
- Genelde **yardımcı (utility/helper)** sınıflarda kullanılır.

---

## ⚖️ 3️⃣ Karşılaştırma Tablosu

|Özellik|Normal Class|Static Class|
|---|---|---|
|Nesne oluşturulabilir mi|✅ Evet (`new`)|❌ Hayır|
|Instance (örnek) değişkeni var mı|✅ Var|❌ Yok|
|Static üyeler barındırabilir mi|✅ Evet|✅ Evet (zaten tüm üyeler static olmalı)|
|Non-static üyeler barındırabilir mi|✅ Evet|❌ Hayır|
|Bellek kullanımı|Her örnek için ayrı|Tek bir kez|
|Kalıtım (inheritance) alabilir mi|✅ Evet|❌ Hayır|
|Kullanım amacı|Nesne tabanlı işlemler|Yardımcı / ortak fonksiyonlar|

---

## 💡 4️⃣ Gerçek hayat örneği

### 🏗️ Normal class örneği:

```csharp
class BankAccount
{
    public string Owner;
    public decimal Balance;

    public void Deposit(decimal amount)
    {
        Balance += amount;
    }
}

```

Burada her kullanıcının (Ali, Ayşe vs.) ayrı hesabı vardır.

Her biri farklı “state” taşır (Balance farklı).

---

### ⚙️ Static class örneği:

```csharp
static class CurrencyConverter
{
    public static decimal UsdToTry(decimal usd)
    {
        return usd * 33.5m;
    }
}

```

Burada bir “nesne durumu” gerekmez.

Sadece **ortak bir hesaplama** yapar — herkes aynı fonksiyonu kullanır.

---

## 🧠 Mülakat cevabı (kısa)

> “Normal sınıflar nesne oluşturularak kullanılır ve her nesnenin kendi verisi vardır.
> 
> Static sınıflar ise örneklenemez, bellekte tek kopyaları bulunur ve genellikle yardımcı metodlar için kullanılır.
> 
> Static class içinde non-static üye bulunmaz ve inheritance yapılamaz.”