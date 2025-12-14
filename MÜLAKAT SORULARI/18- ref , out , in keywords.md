## 🧩 1️⃣ `ref` keyword

**Amaç:**

Bir değişkeni **referansla (adres üzerinden)** metoda göndermektir.

Bu sayede metot içinde yapılan değişiklik **orijinal değişkeni etkiler.**

---

### 💻 Örnek:

```csharp
void DoubleNumber(ref int number)
{
    number = number * 2;
}

int x = 5;
DoubleNumber(ref x);
Console.WriteLine(x); // 10

```

🧠 Açıklama:

- `x` değişkeni **ref ile** gönderildiği için adresi aktarıldı.
- Fonksiyon içindeki değişiklik, `x`’in kendisini değiştirdi.
- `ref` kullanırken değişken **önceden atanmış (initialized)** olmalı.

## 🧩 2️⃣ `out` keyword

**Amaç:**

Bir metoda **boş (başlangıçsız)** değişken göndermek ve

o metottan **geriye değer döndürmek** için kullanılır.

Yani **metot içinde mutlaka bir değer atanmalıdır.**

---

### 💻 Örnek:

```csharp
void GetSquare(int input, out int result)
{
    result = input * input; // out parametresine değer atamak zorundayız
}

int y;
GetSquare(4, out y);
Console.WriteLine(y); // 16

```

🧠 Açıklama:

- `y` önceden değer almasa da olur.
- Metot içinde `out` parametresine **mutlaka değer atanmalıdır**.
- Genellikle birden fazla değer döndürmek için kullanılır.

## 🧩 3️⃣ `in` keyword

**Amaç:**

Bir değişkeni **referansla**, ama **sadece okunabilir (readonly)** şekilde göndermektir.

Metot içinde bu değişken **değiştirilemez.**

---

### 💻 Örnek:

```csharp
void ShowNumber(in int number)
{
    Console.WriteLine(number);
    // number = 10; ❌ Hata: in parametresi değiştirilemez
}

int z = 7;
ShowNumber(in z);

```

🧠 Açıklama:

- `in`, parametreyi **performans için referansla geçirir**,
    
    ama değiştirilememesini garanti eder.
    
- Özellikle **büyük struct** türlerinde (örneğin `struct Point3D`) performans için kullanılır.
    

---

## ⚖️ 4️⃣ Fark Tablosu

|Özellik|`ref`|`out`|`in`|
|---|---|---|---|
|Değişken önceden atanmalı mı?|✅ Evet|❌ Hayır|✅ Evet|
|Metot içinde tekrar atanmak zorunda mı?|❌ Hayır|✅ Evet|❌ Hayır|
|Metot içinde değiştirilebilir mi?|✅ Evet|✅ Evet|❌ Hayır|
|Değer referansla mı gider?|✅ Evet|✅ Evet|✅ Evet|
|Kullanım amacı|Mevcut değeri değiştirmek|Değer döndürmek|Sadece okumak (performans için)|

---

## 💬 Özet (mülakat cevabı gibi):

> “ref, değişkeni referansla gönderir ve değiştirilebilir olmasını sağlar.
> 
> `out`, metoda boş bir değişken gönderip içinde değer atamayı zorunlu kılar.
> 
> `in`, parametreyi referansla ama sadece okunabilir olarak gönderir.”