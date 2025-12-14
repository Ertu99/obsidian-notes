## 🧩 1️⃣ Value Type (Değer Tipi) Nedir?

➡️ **Value type** değişkenler, **verinin kendisini** bellekte tutar.

Bir değişkeni başka birine atarsan, **kopyası** oluşturulur.

---

### 💻 Örnek:

```csharp
int a = 10;
int b = a;  // b = 10 (a’nın kopyası)
b = 20;

Console.WriteLine(a); // 10
Console.WriteLine(b); // 20

```

🧠 Açıklama:

- `a` ve `b` birbirinden tamamen bağımsızdır.
- `b` değiştiğinde `a` etkilenmez.
- Çünkü **değer tipi** bellekte ayrı bir kopya oluşturur.

---

### 📦 Value Type örnekleri:

- `int`, `double`, `bool`, `char`
- `struct`
- `enum`

---

## 🧩 2️⃣ Reference Type (Referans Tipi) Nedir?

➡️ **Reference type** değişkenler, **verinin adresini (referansını)** tutar.

Yani iki değişken aynı nesneyi işaret ederse, biri değiştiğinde diğeri de etkilenir.

## 🧩 Örnek 2 – Reference Type (int’i class içine alalım)

```csharp
class Number
{
    public int Value;
}

Number n1 = new Number();
n1.Value = 10;

Number n2 = n1;  // n2, n1 ile aynı nesneyi işaret ediyor
n2.Value = 20;

Console.WriteLine($"n1.Value = {n1.Value}"); // 20
Console.WriteLine($"n2.Value = {n2.Value}"); // 20

```

🧠 Açıklama:

- Burada `Number` bir **class**, yani **reference type**.
- `n1` ve `n2` aynı adresi (nesneyi) işaret ediyor.
- `n2.Value` değiştiğinde, `n1.Value` da değişiyor.

## ⚖️ Sonuç Karşılaştırması

| Özellik                                 | Value Type (`int`)     | Reference Type (`Number` class) |
| --------------------------------------- | ---------------------- | ------------------------------- |
| Depolama                                | Stack                  | Heap                            |
| Kopyalama                               | Değerin kopyası alınır | Adres (referans) kopyalanır     |
| Birini değiştirince diğeri etkilenir mi | ❌ Hayır                | ✅ Evet                          |
| Örnek Sonuç                             | `a=10, b=20`           | `n1=20, n2=20`                  |

---

💬 **Kısa özet (mülakat cevabı):**

> “int bir value type’tır, biri değişince diğeri etkilenmez.
> 
> Ama aynı int değeri bir class içinde tutarsak, bu artık reference type olur ve iki değişken aynı nesneyi işaret eder.”