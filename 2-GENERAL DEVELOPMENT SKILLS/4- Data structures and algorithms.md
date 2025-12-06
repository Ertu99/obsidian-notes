Bu konu, "Kod yazan kişi" ile "Yazılım Mühendisi" arasındaki farkı belirleyen konudur. Bir backend developer olarak yazdığın kodun **performansını (hızını)** ve **bellek kullanımını (RAM)** yönetmek zorundasın.

Binlerce kullanıcısı olan bir sistemde yanlış veri yapısını seçersen sistem kilitlenir. Doğrusunu seçersen yağ gibi akar.

Roadmap'te geçen yapıları tek tek, **C# dünyasındaki karşılıkları ve çalışma mantıklarıyla** inceleyelim.

---

### 1. Performansın Ölçüsü: Big O Notation (O Notasyonu)

Veri yapılarına girmeden önce, "Neye göre iyi?" sorusunu cevaplamalıyız.

Yazılımda hızı saniye ile ölçmeyiz (çünkü bilgisayarın gücüne göre değişir). Hızı, veri sayısı arttıkça işlemin ne kadar uzadığıyla ölçeriz.

- **O(1) - Constant (Mükemmel):** Veri 1 milyon tane de olsa, 1 tane de olsa erişim süresi aynıdır. (Örn: Dizinin 5. elemanını ver).
    
- **O(log n) - Logarithmic (Harika):** Veri arttıkça süre çok az artar. (Örn: Binary Search / Telefon rehberinde isim aramak).
    
- **O(n) - Linear (Eh İşte):** 10 veri varsa 10 saniye, 1 milyon veri varsa 1 milyon saniye. (Örn: Döngü ile tek tek aramak).
    
- **O(n^2) - Quadratic (Kötü):** İç içe döngüler. Veri arttıkça süre karesi kadar artar. (Örn: Bubble Sort).
    

---

### 2. Linear Structures (Sıralı Yapılar)

#### A. Array (Dizi)

En ilkel ve temel yapıdır. Bellekte yan yana sıralanmış kutucuklardır.

- **Özelliği:** Boyutu **sabittir**. Başta "bana 5 kişilik yer ayır" dersen, 6. kişiyi ekleyemezsin.
    
- **Hız:** Erişim **O(1)**'dir. "3. kutuyu ver" dediğinde anında verir.
    
- **C# Kullanımı:** `int[] sayilar = new int[5];`
    

<iframe width="560" height="315" src="https://www.youtube.com/embed/vOsi1Ra2Uyw?si=COaCs-Zay-sxbJVu" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
#### B. Linked List (Bağlı Liste)

Tren vagonları gibidir. Her eleman bir sonrakinin adresini tutar. Bellekte yan yana olmak zorunda değildir.

- **Özelliği:** Boyutu dinamiktir, istediğin kadar ekle. Ama 5. elemana gitmek için baştan başlayıp tek tek (1->2->3->4->5) gitmen gerekir (**O(n)**).
    
- **C# Kullanımı:** `LinkedList<T>` (C#'ta çok sık kullanılmaz, genelde mülakatlarda sorulur).
    

#### C. List (Dinamik Dizi) - **C#'ın Kralı**

Array'in sabit boyut sorununu çözer. Arka planda Array kullanır ama doldukça otomatik olarak boyutunu **iki katına çıkarır**.

- **C# Kullanımı:** `List<int> sayilar = new List<int>();`
    
- **.NET Developer Notu:** Günlük hayatta %90 bunu kullanacaksın.
    

---

### 3. Key-Value (Anahtar-Değer) Yapıları

#### Hashtable / Dictionary (Sözlük)

Backend dünyasının performans canavarıdır. Verileri bir "Anahtar" (Key) ile saklar.

- **Mantık:** Bir fonksiyon (Hashing Algorithm), anahtarı alır ve bellekteki adresine dönüştürür.
    
- **Hız:** Aradığın veriyi bulmak **O(1)**'dir. Milyonluk veri setinde "ID: 9999 olan kullanıcıyı getir" dersen, `List` ile tek tek aramak yerine `Dictionary` ile eliyle koymuş gibi bulursun.
    
- **C# Kullanımı:** `Dictionary<int, string> users = new Dictionary<int, string>();`
    

---

### 4. Stack & Queue (Yığın ve Kuyruk)

Verilerin geliş-gidiş sırasını yönetmek için kullanılır.

#### A. Stack (Yığın) - LIFO

**Last In First Out (Son giren ilk çıkar).**

- **Örnek:** Üst üste konmuş tabaklar. En son koyduğun tabağı ilk alırsın. Veya tarayıcıdaki "Geri" butonu.
    
- **Metotlar:** `Push()` (ekle), `Pop()` (çıkar).
    
<iframe width="560" height="315" src="https://www.youtube.com/embed/VZ_1I_UuHdE?si=QMRu9syxD_SY836i" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
#### B. Queue (Kuyruk) - FIFO

**First In First Out (İlk giren ilk çıkar).**

- **Örnek:** Ekmek kuyruğu veya yazıcıya gönderilen belgeler. İlk gelen işi önce yaparsın.
    
- **Metotlar:** `Enqueue()` (ekle), `Dequeue()` (çıkar).
    
- **.NET Kullanımı:** RabbitMQ gibi sistemleri öğrenirken bu mantığı kullanacaksın.
    
<iframe width="560" height="315" src="https://www.youtube.com/embed/6L94crb0PgA?si=Wj-zrIQ1rVgnTSLE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
---

### 5. Advanced Structures (İleri Seviye)

Bunları şu an detaylı kodlamana gerek yok ama ne olduğunu bilmelisin:

- **Tree (Ağaç):** Klasör yapısı gibidir. Bir kök (root) ve dallar vardır. HTML DOM yapısı bir ağaçtır.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_T_7fqPfrrk?si=XWJUSccL1BLOoQB8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
- **Graph (Çizge):** Düğümlerin birbirine bağlandığı ağ. Sosyal ağlar (arkadaşlık ilişkileri) veya Haritalar (şehirler ve yollar) graph ile tutulur.
    
<iframe width="560" height="315" src="https://www.youtube.com/embed/CHHwJc1eFw8?si=1oaP3ZF-0f-sGh5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
---

### Pratik: C# ile Performans Farkı

Aşağıdaki C# örneği, `List` ile `Dictionary` arasındaki devasa farkı gösterir.

C#

```csharp
// SENARYO: 1 Milyon kullanıcımız var ve ID'si 999.999 olanı arıyoruz.

// 1. YÖNTEM: List (O(n))
List<User> userList = GetMillionUsers(); 
// Tek tek bakar: Bu mu? Hayır. Bu mu? Hayır... (1 milyon işlem)
var user = userList.FirstOrDefault(u => u.Id == 999999); 

// 2. YÖNTEM: Dictionary (O(1))
Dictionary<int, User> userDict = GetMillionUsersDict();
// Tak diye bulur. (1 işlem)
var user2 = userDict[999999];
```

---

### 6. Algoritmalar (Sıralama ve Arama)

Roadmap'te algoritmalardan da bahsediyor. Bir .NET geliştiricisi olarak Bubble Sort'u oturup elle yazmazsın (C#'ın `List.Sort()` metodu zaten en optimize algoritmayı, genellikle **QuickSort** veya **IntroSort**, kullanır).

Ancak mülakatlar için şunları bil:

1. **Sorting (Sıralama):** Verileri sıraya dizmek. (QuickSort, MergeSort en popülerleridir).
    
2. **Searching (Arama):**
    
    - **Linear Search:** Baştan sona tek tek bakmak (Yavaş).
        
    - **Binary Search:** "Sayı tutmaca" oyunu gibi. Ortaya bak, sayı büyükse sağa git, küçükse sola git. Her adımda ihtimalleri yarıya indirir (Çok hızlı). _Not: Binary Search için listenin sıralı olması şarttır._
        

---

