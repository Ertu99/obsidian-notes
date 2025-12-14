> Access modifier, bir sınıfın, metodun veya özelliğin nerelerden erişilebileceğini (görünebilirliğini) belirler.

Yani kısaca:

> “Bu kodu kim görebilir?” sorusuna verilen cevaptır.

---

## ⚙️ 2️⃣ C#’taki erişim türleri

| Modifier               | Erişim Alanı                                                     | En sık kullanıldığı yer                  |
| ---------------------- | ---------------------------------------------------------------- | ---------------------------------------- |
| **public**             | Her yerden erişilebilir                                          | Sınıflar, metotlar, property’ler         |
| **private**            | Sadece tanımlandığı sınıf içinde                                 | Alan (field), yardımcı metod             |
| **protected**          | Sadece kendi sınıfında **ve kalıtım alan alt sınıflarda**        | Base class → Derived class geçişi        |
| **internal**           | Aynı proje (assembly) içinden erişilebilir                       | Katmanlı yapılarda proje içi paylaşım    |
| **protected internal** | Aynı proje + miras alan sınıflar                                 | Daha geniş erişim isteyen base class’lar |
| **private protected**  | Sadece aynı sınıfta **ve aynı assembly içindeki** alt sınıflarda | Çok özel senaryolar (daha dar erişim)    |

---

## 🧱 3️⃣ Şimdi tek tek görelim

### 🔹 **public**

> Her yerden erişilebilir.
> 
> En açık erişim türüdür.

```csharp
public class Car
{
    public void Drive()
    {
        Console.WriteLine("Car is driving");
    }
}

// Başka sınıfta
Car c = new Car();
c.Drive(); // ✅ Erişilebilir

```

---

### 🔹 **private**

> Sadece tanımlandığı sınıf içinde erişilebilir.
> 
> Dışarıdan **görünmez**.

```csharp
public class Car
{
    private int speed = 0;

    public void Accelerate()
    {
        speed += 10; // ✅ erişim var (aynı class)
    }
}

// Başka sınıfta
Car c = new Car();
// c.speed = 50; // ❌ Hata: speed private

```

🧠 Genelde **kapsülleme (encapsulation)** için kullanılır.

---

### 🔹 **protected**

> Sadece kendi sınıfında ve ondan türeyen (inherit eden) alt sınıflarda erişilebilir.

```csharp
public class Animal
{
    protected void Eat()
    {
        Console.WriteLine("Animal eats");
    }
}

public class Dog : Animal
{
    public void Bark()
    {
        Eat(); // ✅ erişilebilir (miras aldığı için)
    }
}

// Başka sınıfta
Animal a = new Animal();
// a.Eat(); // ❌ Hata

```

🧠 `protected` genelde **base class → derived class** iletişimi içindir.

---

### 🔹 **internal**

> Sadece aynı proje (assembly) içinden erişilebilir.
> 
> Başka projeden erişilemez.

```csharp
internal class Helper
{
    public static void SayHi() => Console.WriteLine("Hi!");
}

// Aynı proje içinden ✅
Helper.SayHi();

// Farklı proje (örneğin başka .dll) ❌ Erişilemez

```

🧠 Büyük projelerde “Bu class sadece bu modül tarafından kullanılabilir” demek için kullanılır.

---

### 🔹 **protected internal**

> Hem protected hem internal davranır:

- Aynı proje içinden her yerden erişilebilir.
- - Başka projede olsan bile **kalıtım aldıysan** erişebilirsin.

```csharp
public class Base
{
    protected internal void Speak() => Console.WriteLine("Hello!");
}

public class Derived : Base
{
    public void Test() => Speak(); // ✅ (inherit)
}

```

---

### 🔹 **private protected**

> Yalnızca:

- **Aynı assembly (proje)** içindeki
- **Alt sınıflardan (inherit eden)** erişilebilir.

Daha kısıtlı versiyondur.

```csharp
public class Base
{
    private protected void OnlyHere() => Console.WriteLine("Restricted");
}

public class Derived : Base
{
    public void Test() => OnlyHere(); // ✅
}

// Farklı proje → ❌ erişemez

```

---

## 🧠 4️⃣ Özet Tablo

|Modifier|Aynı Sınıf|Alt Sınıf (inherit)|Aynı Assembly|Farklı Assembly|
|---|---|---|---|---|
|**public**|✅|✅|✅|✅|
|**private**|✅|❌|❌|❌|
|**protected**|✅|✅|❌|❌|
|**internal**|✅|✅|✅|❌|
|**protected internal**|✅|✅|✅|✅ (sadece inherit eden)|
|**private protected**|✅|✅ (aynı projede)|✅|❌|

---

## 🧩 5️⃣ Gerçek hayat örneği

### 🔹 Örneğin bir `Car` sınıfı düşün:

```csharp
public class Car
{
    public string Model { get; set; } = "";   // Herkes görebilir
    private int Speed { get; set; } = 0;      // Sadece Car içinde
    protected void Accelerate() { Speed += 10; } // Miras alanlar görebilir
}

```

### 🔹 Alt sınıf:

```csharp
public class SportsCar : Car
{
    public void NitroBoost()
    {
        Accelerate(); // ✅ protected erişimi
    }
}

```

### 🔹 Dışarıdan:

```csharp
Car c = new Car();
c.Model = "BMW"; // ✅
 // c.Accelerate(); // ❌ Hata, protected

```

---

## 💬 6️⃣ Mülakat cevabı (kısa ve net)

> “C#’ta erişim belirleyiciler, bir üyenin nerelerden erişilebileceğini belirler.
> 
> `public` her yerden erişilebilir, `private` sadece tanımlandığı sınıfta,
> 
> `protected` miras alan sınıflarda, `internal` aynı projede,
> 
> `protected internal` ikisinin birleşimidir,
> 
> `private protected` ise en kısıtlı halidir.”