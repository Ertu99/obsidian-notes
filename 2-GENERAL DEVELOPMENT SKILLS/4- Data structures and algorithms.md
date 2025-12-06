

## Giriş

Yazılım mühendisliğinin temel disiplinlerinden biri olan Veri Yapıları ve Algoritmalar (DSA), modern uygulama geliştirme süreçlerinde, özellikle.NET Core ve ardıl sürümleri (.NET 5/6/7/8+) gibi yönetilen (managed) çalışma zamanına sahip platformlarda kritik bir rol oynamaktadır. Bu rapor, "Big O" notasyonu ile ifade edilen teorik karmaşıklık analizinden başlayarak,.NET çalışma zamanının (Common Language Runtime - CLR) bellek yönetimi prensiplerine, ilkel veri yapılarının (diziler, listeler) iç mimarisine, karmaşık ağaç ve grafik yapılarına ve son olarak modern.NET'in sunduğu yüksek performanslı bellek erişim modellerine (`Span<T>`, `Memory<T>`) kadar uzanan geniş bir spektrumu kapsamaktadır.

Raporun temel amacı, okuyucuya sadece bir veri yapısının nasıl kullanılacağını öğretmek değil, o yapının "kaputun altında" nasıl çalıştığını, bellek (RAM) ve işlemci (CPU) ile nasıl etkileşime girdiğini ve Çöp Toplayıcı (Garbage Collector - GC) üzerinde yarattığı baskıyı mikroskobik düzeyde analiz etme yetkinliği kazandırmaktır. Bir.NET geliştiricisi için `List<T>` ile `LinkedList<T>` arasındaki tercih, sadece eleman ekleme hızının teorik karşılaştırması değil, aynı zamanda işlemci önbelleği (CPU Cache) yerelliği, bellek sayfalama (paging) ve nesne referans maliyetlerinin de bir fonksiyonudur. Bu çalışma, akademik teoriyi endüstriyel pratikle birleştirerek, ölçeklenebilir ve yüksek performanslı.NET uygulamaları geliştirmek isteyen mühendisler için başvuru kaynağı niteliğindedir.

## Bölüm 1: Algoritmik Karmaşıklık Analizi ve Teorik Temeller

Yazılım sistemlerinin performansını değerlendirmek ve karşılaştırmak için kullanılan evrensel dil, asimptotik analiz veya yaygın bilinen adıyla Big O notasyonudur. Bu notasyon, bir algoritmanın girdi boyutu ($n$) arttıkça çalışma süresinin veya bellek gereksiniminin nasıl değiştiğini (büyüme hızını) matematiksel olarak modeller.

### 1.1 Asimptotik Notasyonlar: O, $\Omega$ ve $\Theta$

Geliştiriciler arasında genellikle sadece "Big O" ($O$) konuşulsa da, tam bir analiz için üç temel notasyonun anlaşılması gerekir:

- **Big O ($O$ - Üst Sınır):** Algoritmanın "en kötü durum" (worst-case) senaryosunu temsil eder. Örneğin, $O(n^2)$ olan bir algoritma, girdi boyutu ne olursa olsun karesel büyüme hızını aşmayacaktır. Sistem mühendisliğinde darboğazları belirlemek için en kritik ölçüttür.4
    
- **Big Omega ($\Omega$ - Alt Sınır):** Algoritmanın "en iyi durum" (best-case) senaryosunu ifade eder. Sıralı bir dizide arama yaparken ilk elemanı bulmak $\Omega(1)$ karmaşıklığındadır. Ancak sistemler en iyi duruma göre tasarlanmaz.
    
- **Big Theta ($\Theta$ - Kesin Sınır):** Algoritmanın hem alt hem de üst sınırının aynı fonksiyonla ifade edilebildiği durumdur. Bir algoritmanın her durumda (best, average, worst) benzer performans gösterdiğini belirtir.4
    

### 1.2 Zaman Karmaşıklığı Sınıfları ve.NET Örnekleri

Algoritmaların zaman karmaşıklığı, işlemci döngülerinin tüketim hızını belirler..NET Framework içerisindeki veri yapıları üzerinden bu sınıfları örneklendirebiliriz:

- **$O(1)$ - Sabit Zaman (Constant Time):** Girdi boyutundan bağımsızdır. `List<T>` koleksiyonunda indekse dayalı erişim (`list[1]`) veya `Dictionary<TKey, TValue>` yapısında anahtar ile veri çekmek (çarpışmasız durumda) bu kategoriye girer. Bu, performansın "altın standardıdır".5
    
- **$O(\log n)$ - Logaritmik Zaman (Logarithmic Time):** Girdi boyutu her adımda belirli bir oranda (genellikle yarıya) azalır. Sıralı bir dizide İkili Arama (Binary Search) veya `SortedDictionary` gibi dengeli ağaç yapılarında eleman arama işlemleri logaritmik karmaşıklığa sahiptir. 1 milyon elemanlı bir veri setinde arama yapmak sadece yaklaşık 20 adım sürer ($\log_2 10^6 \approx 20$).3
    
- **$O(n)$ - Doğrusal Zaman (Linear Time):** İşlem süresi girdi boyutuyla birebir artar. `List<T>.Contains()` metodu, aranan elemanı bulmak için tüm listeyi gezmek zorunda kalabilir. Döngüler (`foreach`, `for`) genellikle bu karmaşıklığı üretir.2
    
- **$O(n \log n)$ - Doğrusal-Logaritmik Zaman (Linearithmic Time):** Verimli sıralama algoritmalarının (Merge Sort, Heap Sort, Quick Sort) standart karmaşıklığıdır..NET'in `Array.Sort` veya LINQ'in `OrderBy` metotları bu sınıfta çalışır. Büyük veri setlerini sıralamak için kabul edilebilir üst sınırdır.8
    
- **$O(n^2)$ - Karesel Zaman (Quadratic Time):** İç içe döngüler (nested loops) içeren algoritmalar bu sınıfa girer. Kabarcık Sıralaması (Bubble Sort) gibi verimsiz algoritmalar veya iki listeyi iç içe `for` ile karşılaştırmak karesel karmaşıklık yaratır. Girdi boyutu 10 kat arttığında süre 100 kat artar, bu da ölçeklenebilirlik için büyük bir tehdittir.5
    

### 1.3 Alan Karmaşıklığı (Space Complexity)

Alan karmaşıklığı, algoritmanın çalışması sırasında ihtiyaç duyduğu "ekstra" belleği ifade eder. Girdinin kendisi bu hesaba dahil edilmez..NET ortamında alan karmaşıklığı, doğrudan Garbage Collector (GC) performansı ile ilişkilidir. Örneğin, `Quick Sort` yerinde (in-place) sıralama yaptığı için $O(\log n)$ (rekürsif yığın çağrıları için) alan kullanırken, `Merge Sort` dizinin bir kopyasını oluşturduğu için $O(n)$ ek alan gerektirir.2 Yüksek bellek kullanımı, GC'nin daha sık çalışmasına (GC Pressure) ve uygulamanın duraklamasına (Latency) neden olabilir.

## Bölüm 2:.NET Çalışma Zamanı (Runtime) Bellek Mimarisi

.NET Core üzerinde veri yapılarının davranışını tam olarak kavramak için, çalışma zamanının belleği nasıl yönettiğini, Yığın (Stack) ve Öbek (Heap) kavramlarını ve Veri Tiplerini derinlemesine incelemek gerekir.

### 2.1 Stack ve Heap: Bellek Tahsisinin Fiziği

.NET bellek yönetimi iki ana bölgeye ayrılır:

1. **Stack (Yığın):** Çok hızlıdır, LIFO (Last-In, First-Out) mantığıyla çalışır ve işlemci önbelleği (L1/L2 Cache) ile son derece uyumludur. Metot çağrıları, yerel değişkenler ve değer tipleri burada saklanır. Stack üzerindeki bellek yönetimi deterministiktir; bir metot bittiğinde (scope dışına çıkıldığında) bellek anında serbest bırakılır.10

<iframe width="560" height="315" src="https://www.youtube.com/embed/VZ_1I_UuHdE?si=Q6eLk-m5BoURLmqx" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

1. **Heap (Öbek):** Daha büyük ve kaotik bir bellek alanıdır. Referans tipleri (`class`, `interface`, `delegate`, `string`, `array`) burada yaşar. Heap üzerindeki nesnelerin yaşam döngüsü Garbage Collector (GC) tarafından yönetilir. Heap'ten bellek tahsis etmek (allocation), Stack'e göre daha maliyetlidir çünkü GC'nin boş bir blok bulması ve yönetmesi gerekir.10

<iframe width="560" height="315" src="https://www.youtube.com/embed/OVTXS2YlpnQ?si=jFmI5OYipZj_mfHJ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    

### 2.2 Değer Tipleri ve Referans Tipleri

.NET veri tipleri, bellekte saklanma biçimlerine göre ikiye ayrılır ve bu ayrım performans üzerinde belirleyicidir:

- **Değer Tipleri (Value Types - `struct`, `enum`):** Verinin kendisini tutarlar. Stack üzerinde (veya başka bir nesnenin içinde gömülü olarak) saklanırlar. Kopyalandıklarında değerin tamamı kopyalanır. Bellek tahsisi ve erişimi çok hızlıdır.
    
- **Referans Tipleri (Reference Types - `class`):** Verinin bellekteki adresini (pointer) tutarlar. Verinin kendisi Heap'te, adresi ise Stack'te durur. Kopyalandıklarında sadece adres kopyalanır, veri paylaşılır.
    

Veri Yapısı Seçiminde Kritik Karar:

Küçük ve değişmez veriler (örneğin Point(x, y), ComplexNumber) için struct kullanmak, Heap tahsisini ve GC yükünü ortadan kaldırır. Ancak büyük struct'ları metotlara parametre olarak geçmek, verinin sürekli kopyalanmasına neden olacağı için performansı düşürebilir. Bu durumda ref anahtar kelimesi ile referans geçişi sağlanmalıdır.10

### 2.3 Boxing ve Unboxing Maliyetleri

Bir değer tipinin (örneğin `int`), bir referans tipine (örneğin `object` veya bir `interface`) dönüştürülmesi işlemine **Boxing** denir. Bu işlem sırasında:

1. Heap üzerinde yeni bir nesne (kutu) tahsis edilir.
    
2. Değer Stack'ten alınıp bu kutunun içine kopyalanır.
    
3. Kutunun referansı geri döndürülür.
    

**Unboxing** ise bu işlemin tersidir. Boxing işlemi, hem bellek tahsisi hem de kopyalama gerektirdiği için hesaplama açısından pahalıdır ($O(1)$ olsa da sabit katsayısı yüksektir). `ArrayList` gibi eski, jenerik olmayan (non-generic) koleksiyonlar, her eleman için Boxing uyguladığından modern `List<T>`'ye göre çok daha yavaştır. Jenerik mimari (`List<int>`), değer tiplerini Boxing olmadan doğrudan saklayarak bu maliyeti ortadan kaldırır.11

## Bölüm 3: Bitişik Bellek Yapıları: Diziler ve Listeler

### 3.1 Diziler (Arrays): Sistemin Temel Taşı

Diziler,.NET'teki en temel ve en hızlı veri yapısıdır. Bellekte kesintisiz (contiguous) bir blok olarak tahsis edilirler. Bu yapı, modern işlemcilerin "Spatial Locality" (Uzamsal Yerellik) prensibine mükemmel uyum sağlar. İşlemci, dizinin bir elemanına eriştiğinde, donanım seviyesindeki öngetiriciler (hardware prefetchers) sonraki elemanları da önbelleğe (CPU Cache) yükler. Bu sayede, dizi üzerinde döngü kurmak (`for`, `foreach`), bağlı listelere veya dağınık nesnelere göre katbekat daha hızlıdır.14 Dizilerin boyutu oluşturulduğu anda sabitlenir ve çalışma zamanında değiştirilemez.

### 3.2 `List<T>`: Dinamik Dizi Mühendisliği

Geliştiricilerin en sık kullandığı koleksiyon olan `List<T>`, aslında "dinamik boyutlu dizi" deseninin (pattern) bir implementasyonudur. `System.Collections.Generic` altında bulunan bu sınıf, arka planda `T _items` adında bir dizi yönetir.16

#### İç Çalışma Mekanizması ve Kapasite Yönetimi

Bir `List<T>` oluşturulduğunda, genellikle küçük bir başlangıç kapasitesine (örneğin 4 veya 0) sahip bir dizi tahsis edilir. Kullanıcı `Add(item)` metodu ile eleman eklediğinde şu mantık işler:

1. **Kontrol:** Dahili dizide (`_items`) boş yer var mı? (`Count < Capacity`)
    
2. **Ekleme:** Yer varsa, eleman sıradaki boş indekse eklenir ve `Count` artırılır. Bu işlem $O(1)$'dir.
    
3. **Yeniden Boyutlandırma (Resizing):** Eğer dizi doluysa, `EnsureCapacity` metodu tetiklenir:
    
    - Mevcut kapasitenin iki katı büyüklüğünde yeni bir dizi tahsis edilir (örneğin 4'ten 8'e, 8'den 16'ya).
        
    - `Array.Copy` ile eski dizideki tüm veriler yeni diziye kopyalanır.
        
    - Eski dizi serbest bırakılır.
        
    - Yeni eleman eklenir.17
        

Bu yeniden boyutlandırma işlemi $O(n)$ maliyetindedir. Ancak bu işlem nadiren gerçekleştiği için (logaritmik sıklıkta), `Add` işleminin "amortize edilmiş" (amortized) karmaşıklığı $O(1)$ olarak kabul edilir. Yine de, milyonlarca eleman eklenecek bir listede, dizinin defalarca yeniden boyutlandırılması ve kopyalanması ciddi performans kaybı ve GC baskısı yaratır.

**Performans İpucu:** Eleman sayısını yaklaşık olarak biliyorsanız, listeyi oluştururken kapasiteyi belirtmek (`new List<int>(10000)`) bu maliyetleri sıfıra indirir.17

#### `TrimExcess`: Bellek Tasarrufu

Büyük bir liste oluşturup ardından elemanların çoğunu sildiğinizde, listenin `Capacity` değeri (dahili dizinin boyutu) otomatik olarak küçülmez. `List<T>.TrimExcess()` metodu, eğer listenin doluluk oranı %90'ın altındaysa, dahili diziyi mevcut eleman sayısına (`Count`) eşit olacak şekilde yeniden boyutlandırır. Bu işlem de $O(n)$ maliyetindedir (kopyalama gerektirir) ve sadece belleğin kritik olduğu durumlarda kullanılmalıdır.21

## Bölüm 4: Bağlantılı Veri Yapıları ve Önbellek Sorunsalı

### 4.1 `LinkedList<T>`: Teorik Avantajlar ve Pratik Gerçekler

.NET'teki `LinkedList<T>`, çift yönlü bağlı liste (doubly linked list) implementasyonudur. Her eleman (`LinkedListNode<T>`), verinin kendisini, bir önceki düğüme (`Previous`) ve bir sonraki düğüme (`Next`) olan referansları tutan bir nesnedir.24

Teorik olarak, bağlı listenin arasına eleman eklemek veya çıkarmak $O(1)$ karmaşıklığındadır (sadece pointer'ları güncellemek yeterlidir). Oysa `List<T>` yapısında araya ekleme yapmak, o noktadan sonraki tüm elemanların kaydırılmasını gerektirdiği için $O(n)$ maliyetindedir.
<iframe width="560" height="315" src="https://www.youtube.com/embed/QAC4JDcwZo4?si=hYEImWom_DaXDNIa" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
### 4.2 Önbellek Lokalitesi (Cache Locality) Analizi

Kağıt üzerindeki bu $O(1)$ avantajı, modern donanımlarda çoğu zaman yanıltıcıdır. `LinkedList<T>` düğümleri Heap üzerinde rastgele bellek adreslerinde oluşturulur. Bir düğümden diğerine geçmek ("pointer chasing"), işlemcinin sürekli olarak farklı bellek sayfalarına erişmesini gerektirebilir.

- **List (Dizi):** Veriler bitişiktir. İşlemci bir veri bloğunu önbelleğe çektiğinde, sonraki veriler de gelmiş olur. Önbellek ıska oranı (Cache Miss Rate) çok düşüktür.
    
- **LinkedList:** Veriler dağınıktır. Her düğüm erişimi, potansiyel bir önbellek ıskalaması ve ana belleğe (RAM) maliyetli bir yolculuk demektir. RAM erişimi, önbellek erişiminden yüzlerce kat daha yavaştır.14
    

Yapılan benchmark testlerinde, milyonlarca elemanlı bir `List<T>` üzerinde araya ekleme yapmak (veri kaydırma maliyetine rağmen), genellikle `LinkedList<T>` üzerinde gezerek (traverse) ekleme yapmaktan daha hızlı sonuçlanmaktadır. `LinkedList<T>` kullanımı,.NET dünyasında sadece çok spesifik senaryolarla (örneğin, eleman referansının elde tutulduğu ve sürekli $O(1)$ silme/ekleme yapılan LRU Cache implementasyonları gibi) sınırlı kalmalıdır.24

## Bölüm 5: Hash Tabanlı Veri Yapıları ve `Dictionary` Mimarisi

### 5.1 Hash Tablosu Teorisi ve `Dictionary<TKey, TValue>`

`Dictionary<TKey, TValue>`, anahtar-değer çiftlerini saklayan ve anahtara göre ortalama $O(1)$ erişim hızı sunan bir veri yapısıdır..NET implementasyonu, "Separate Chaining" (Ayrık Zincirleme) yönteminin optimize edilmiş bir varyasyonunu kullanır.26

#### İç Veri Yapıları

Dictionary, verileri yönetmek için dahili olarak iki ana dizi kullanır:

1. **`int buckets`:** Hash kodunun modülo işlemi ile haritalandığı kova indekslerini tutar. Bu dizi, zincirin başlangıç noktasını işaret eder.
    
2. **`Entry entries`:** Verinin kendisinin saklandığı yapıdır. `Entry` struct'ı şu alanlara sahiptir:
    
    - `hashCode`: Anahtarın hesaplanmış hash kodu (tekrar hesaplamayı önlemek için saklanır).
        
    - `next`: Aynı kovaya düşen bir sonraki elemanın indeksi (çarpışma yönetimi için).
        
    - `key`: Anahtar.
        
    - `value`: Değer.26
        

### 5.2 Çarpışma Çözümlemesi (Collision Resolution) ve Zincirleme

İki farklı anahtarın hash kodu, bucket dizisinin boyutuna göre mod alındığında aynı indeksi üretebilir. Buna "Hash Çarpışması" (Collision) denir..NET, bu durumu çözmek için `entries` dizisi içinde sanal bir bağlı liste oluşturur.

Bir çarpışma olduğunda:

- Yeni veri `entries` dizisindeki bir sonraki boş slota yazılır.
    
- Yeni `Entry`'nin `next` alanı, o bucket'ta daha önce bulunan elemanın indeksine ayarlanır.
    
- buckets dizisi, yeni elemanın indeksini gösterecek şekilde güncellenir.
    
    Böylece aynı bucket'a düşen elemanlar birbirine next indeksleri üzerinden bağlanmış olur. Aranan bir anahtar için, önce bucket bulunur, sonra next indeksleri takip edilerek zincir (chain) taranır. Bu tarama işlemi doğrusal ($O(n)$) olsa da, iyi bir hash fonksiyonu ile zincirler çok kısa kalır ve ortalama erişim $O(1)$ olur.26
    

### 5.3 `GetHashCode` ve Performans Tuzakları

Dictionary performansının kalbi `GetHashCode` metodudur.

- **Kötü Hash Fonksiyonu:** Eğer özel bir sınıf için yazdığınız `GetHashCode` metodu her zaman sabit bir değer (örneğin `return 1;`) dönerse, tüm elemanlar aynı bucket'a düşer. Dictionary, devasa bir bağlı listeye dönüşür ve arama hızı $O(1)$'den $O(n)$'e düşer.
    
- **Değişkenlik (Mutability):** Dictionary'e anahtar olarak eklenen bir nesnenin, hash kodunu etkileyen alanları sonradan değiştirilmemelidir. Hash kodu değişirse, nesne yanlış bucket'ta kalır ve bir daha bulunamaz.30
    
- **HashDoS Saldırısı:** Kötü niyetli kullanıcılar, aynı hash kodunu üreten binlerce anahtar göndererek (Collision Attack), sunucunun CPU'sunu $O(n)$ arama maliyetiyle kilitleyebilir..NET Core, bu saldırıları önlemek için `string` hash hesaplamalarında her uygulama başlatıldığında rastgele belirlenen bir "tohum" (seed) kullanır.31
    

**Modern Yöntem:**.NET Core ve sonrası için `GetHashCode` implementasyonlarında `HashCode.Combine(prop1, prop2,...)` statik metodu kullanılmalıdır. Bu metod, bit kaydırma ve karıştırma işlemlerini optimize edilmiş bir algoritma ile yapar.32

## Bölüm 6: Yığınlar, Kuyruklar ve Öncelik Yönetimi

### 6.1 `Stack<T>` ve `Queue<T>`

Bu iki veri yapısı da arka planda dizi (`T`) kullanır ve `List<T>` gibi dinamik olarak büyür.

- **Stack (LIFO):** Son giren ilk çıkar. DFS algoritmalarında, geri alma (undo) işlemlerinde ve sözdizimi analizinde (parsing) kullanılır.

<iframe width="560" height="315" src="https://www.youtube.com/embed/VZ_1I_UuHdE?si=Q6eLk-m5BoURLmqx" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

- **Queue (FIFO):** İlk giren ilk çıkar. BFS algoritmalarında, mesaj kuyruklarında ve işlem sıraya almada kullanılır. `Queue<T>`, performansı artırmak için "Dairesel Tampon" (Circular Buffer) mantığıyla çalışır. Dizinin başından eleman silindiğinde (Dequeue), diğer elemanlar kaydırılmaz; sadece "baş" (head) ve "kuyruk" (tail) indeksleri güncellenir. Bu sayede hem ekleme hem çıkarma $O(1)$ karmaşıklığındadır.35

<iframe width="560" height="315" src="https://www.youtube.com/embed/6L94crb0PgA?si=AsiESzf1UeqF4HrZ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### 6.2 `PriorityQueue<TElement, TPriority>` (.NET 6+)

Uzun yıllar boyunca.NET geliştiricileri, öncelik kuyruğu ihtiyacı için üçüncü parti kütüphaneler kullanmak veya kendi Heap yapılarını yazmak zorundaydı..NET 6 ile birlikte, yüksek performanslı bir `PriorityQueue` sınıfı BCL'e eklendi.

Mimari Detay: Dörtlü Min-Yığın (Quaternary Min-Heap)

Standart "Binary Heap" (İkili Yığın) yapısında her düğümün 2 çocuğu varken,.NET PriorityQueue implementasyonu "Dörtlü Yığın" ($d=4$-ary heap) kullanır. Yani her düğümün 4 çocuğu vardır.

- **Neden 4 Çocuk?** Ağacın derinliğini azaltmak ve bellek yerelliğini (cache locality) artırmak için. Daha sığ bir ağaç, kökten yaprağa inerken daha az karşılaştırma ve bellek erişimi demektir. Modern CPU önbellek hatları, bitişik duran 4 çocuğu tek seferde yükleyebilir, bu da performansı artırır.
    
- **Karmaşıklık:** Ekleme (`Enqueue`) ve en yüksek öncelikli elemanı çekme (`Dequeue`) işlemleri $O(\log_4 n)$ karmaşıklığındadır.37
    

## Bölüm 7: Ağaçlar ve Sıralı Veri Yapıları

Verilerin belirli bir düzende (genellikle anahtara göre sıralı) saklanması gerektiğinde, Hash tabanlı yapılar yetersiz kalır (çünkü hash sırasızdır). Burada ağaç tabanlı yapılar devreye girer.
<iframe width="560" height="315" src="https://www.youtube.com/embed/_T_7fqPfrrk?si=8RdyZH1-1I6HgaRR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### 7.1 `SortedDictionary` vs `SortedList`: Ezeli Rekabet

Her iki sınıf da `IDictionary` arayüzünü uygular ve elemanları anahtara göre sıralı tutar, ancak iç yapıları ve performans profilleri taban tabana zıttır.

|**Özellik**|**SortedDictionary<TKey, TValue>**|**SortedList<TKey, TValue>**|
|---|---|---|
|**Veri Yapısı**|Kırmızı-Siyah Ağaç (Red-Black Tree)|İki adet Sıralı Dizi (`Keys`, `Values`)|
|**Bellek Kullanımı**|Yüksek (Her düğüm için nesne + referanslar)|Düşük (Sadece veri dizileri)|
|**Ekleme/Silme**|$O(\log n)$ (Ağaç rotasyonu, hızlı)|$O(n)$ (Dizi kaydırma, yavaş)|
|**Arama (Lookup)**|$O(\log n)$ (Ağaç gezme)|$O(\log n)$ (İkili Arama - Binary Search)|
|**İterasyon**|Daha yavaş (Bellekte dağınık)|Çok hızlı (Bellekte bitişik - Cache friendly)|
|**Kullanım Senaryosu**|Sık sık yazma/silme yapılan durumlar|Veri bir kez yüklenip çok kez okunacaksa|

**Detaylı Analiz:**

- **SortedDictionary:** Dengeli bir İkili Arama Ağacı (BST) türü olan Kırmızı-Siyah Ağaç kullanır. Bu yapı, ağacın derinliğinin her zaman logaritmik kalmasını garanti eder. Ekleme sırasında ağaç dengesi bozulursa, "rotasyon" işlemleriyle denge sağlanır. Pointer manipülasyonu olduğu için büyük veri setlerinde ekleme işlemi `SortedList`'ten çok daha hızlıdır.40
    
- **SortedList:** Arka planda diziler tuttuğu için, araya bir eleman eklemek tüm dizinin kaydırılmasını gerektirir ($O(n)$). Ancak bellek kullanımı çok daha azdır ve CPU önbelleğini çok verimli kullanır. Küçük ve orta ölçekli veri setlerinde veya "Write-Once, Read-Many" (Bir yaz, çok oku) senaryolarında `SortedDictionary`'den daha performanslı olabilir.40
    

## Bölüm 8: Sıralama ve Arama Algoritmaları

### 8.1 `Array.Sort` ve "Introsort" Mucizesi

.NET'in `Array.Sort` veya `List.Sort` metotları, tek bir algoritma kullanmak yerine, duruma göre strateji değiştiren hibrit bir algoritma olan **Introsort (Introspective Sort)** kullanır.

1. **Başlangıç (QuickSort):** Algoritma, genellikle en hızlı olan QuickSort ile başlar. Pivot seçimi ve bölümleme (partitioning) yapılır. Ortalama karmaşıklık $O(n \log n)$'dir.
    
2. **Gözlem (Introspection):** Eğer QuickSort'un özyineleme (recursion) derinliği $2 \log n$ seviyesini aşarsa, bu durum "kötü pivot seçimi" yapıldığını ve algoritmanın $O(n^2)$ felaket senaryosuna doğru gittiğini gösterir.
    
3. **Strateji Değişikliği (HeapSort):** Derinlik sınırı aşıldığında,.NET hemen QuickSort'u bırakır ve kalan alt dizi için HeapSort'a geçer. HeapSort, en kötü durumda bile $O(n \log n)$ garantisi verir.
    
4. **Mikro Optimizasyon (Insertion Sort):** Bölümlenen parçaların boyutu 16 elemanın altına düştüğünde, bu küçük parçalar için Insertion Sort (Araya Ekleme Sıralaması) kullanılır. Küçük dizilerde Insertion Sort, düşük sabit maliyetleri (low overhead) nedeniyle QuickSort'tan daha hızlıdır.43
    

Bu hibrit yapı,.NET sıralama fonksiyonlarının hem çok hızlı hem de "kötü durum" senaryolarına karşı güvenli olmasını sağlar.

### 8.2 İkili Arama (Binary Search)

Sıralı bir `List<T>` veya dizi üzerinde `BinarySearch` metodu kullanıldığında, "Böl ve Yönet" prensibiyle arama yapılır. Ortadaki elemana bakılır, aranan değer büyükse sağ taraf, küçükse sol taraf seçilir ve işlem tekrarlanır. Karmaşıklık $O(\log n)$'dir. 1 milyar elemanlı sıralı bir listede, aranan değeri bulmak en fazla 30 adım sürer. `List<T>.Contains` ($O(n)$) ile kıyaslandığında devasa bir performans farkı yaratır.8

<iframe width="560" height="315" src="https://www.youtube.com/embed/_T_7fqPfrrk?si=8RdyZH1-1I6HgaRR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Bölüm 9: Grafikler (Graphs) ve Temsil Yöntemleri

.NET BCL içinde yerleşik bir `Graph` sınıfı yoktur, ancak geliştiriciler ihtiyaçlarına göre grafikleri iki temel yöntemle modeller.

<iframe width="560" height="315" src="https://www.youtube.com/embed/CHHwJc1eFw8?si=9mA8cV1rU-0kCZVW" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### 9.1 Komşuluk Listesi (Adjacency List) vs. Matrisi (Adjacency Matrix)

- **Komşuluk Listesi:** Her düğüm, kendisine bağlı olan komşu düğümlerin bir listesini tutar.
    
    C#
    
    ```
    Dictionary<int, List<int>> graph = new();
    ```
    
    Bu yöntem **Seyrek Grafikler (Sparse Graphs)** için idealdir. Gerçek hayattaki çoğu grafik (sosyal ağlar, yol haritaları, internet) seyrektir (her düğüm herkese bağlı değildir). Bellek kullanımı $O(V + E)$ (Düğüm + Kenar) oranındadır. Komşuları gezmek hızlıdır.35
    
- Komşuluk Matrisi: İki boyutlu bir dizi kullanılır (int[,] matrix). matrix[i, j] = 1 ise i ve j düğümleri arasında bağlantı vardır.
    
    Bu yöntem Yoğun Grafikler (Dense Graphs) için veya iki düğüm arasında bağlantı olup olmadığını $O(1)$ sürede kontrol etmek gerektiğinde kullanılır. Ancak bellek kullanımı $O(V^2)$ olduğundan, büyük grafiklerde ciddi bellek israfına yol açar.48
    

### 9.2 Gezinme Algoritmaları (BFS ve DFS)

- **BFS (Breadth-First Search - Sığ Öncelikli Arama):** Grafikte katman katman ilerler. Başlangıç düğümüne en yakın düğümleri önce ziyaret eder. **En kısa yol** problemlerinde (ağırlıksız grafiklerde) kullanılır..NET'te `Queue<T>` kullanılarak implemente edilir.35
    
- **DFS (Depth-First Search - Derin Öncelikli Arama):** Bir yolu sonuna kadar takip eder, gidecek yer kalmadığında geri döner (backtracking). Labirent çözme, topolojik sıralama ve döngü tespiti problemlerinde kullanılır..NET'te `Stack<T>` veya özyineleme (recursion) ile implemente edilir.
    

## Bölüm 10: Modern.NET ve Yüksek Performanslı Bellek Yönetimi

.NET Core 2.1 ile başlayan ve.NET 5/6/7/8 ile olgunlaşan dönemde, "Zero Allocation" (Sıfır Tahsis) ve yüksek performanslı bellek erişimi için yeni türler tanıtıldı. Bu türler, özellikle yüksek trafikli web sunucuları, IoT ve oyun geliştirme gibi alanlarda DSA yaklaşımını değiştirdi.

### 10.1 `Span<T>` ve `Memory<T>` Devrimi

Geleneksel olarak, bir `string`'in bir parçasını almak (`Substring`) veya bir dizinin bir bölümünü kopyalamak, Heap üzerinde yeni bir nesne tahsisini ve veri kopyalamayı gerektirirdi. `Span<T>`, belleğin (Heap, Stack veya Unmanaged) herhangi bir bitişik bölgesine "pencere" açan, struct tabanlı bir türdür.

- **Tahsisiz Dilimleme (Slicing):** `Span<T>.Slice(start, length)` metodu, veriyi kopyalamaz; sadece bellekteki o bölgeye işaret eden yeni bir `Span` (pointer + length) oluşturur. Bu işlem $O(1)$'dir ve Heap tahsisi yapmaz. String işleme (parsing) algoritmalarında `Substring` yerine `AsSpan().Slice()` kullanmak, GC yükünü ortadan kaldırır ve performansı katlar.50
    
- **`ref struct` Kısıtlamaları:** `Span<T>` bir `ref struct` olduğu için sadece Stack üzerinde yaşayabilir. Bu nedenle `class` içinde alan (field) olamaz, `async` metotlarda `await` sonrası kullanılamaz ve kutulanamaz (boxing yapılamaz).
    
- **`Memory<T>`:** `Span`'ın Heap üzerinde saklanabilen kuzenidir. `async` işlemler veya uzun ömürlü nesneler içinde veri dilimlerini tutmak için kullanılır, ancak erişim sırasında `Span`'a dönüştürülmesi gerekir.52
    

### 10.2 `ArrayPool<T>`: Dizi Havuzlama

Büyük dizilerin ($>85KB$) sürekli oluşturulup yok edilmesi, bu dizilerin "Large Object Heap" (LOH) bölgesine gitmesine ve bellek parçalanmasına (fragmentation) yol açar. `System.Buffers.ArrayPool<T>`, dizileri tekrar kullanmak için bir havuz mekanizması sunar.

C#

	```csharp
// Kiralama
var buffer = ArrayPool<byte>.Shared.Rent(4096); 
try {
    // İşlemler... (buffer boyutu 4096'dan büyük olabilir!)
} finally {
    // İade
    ArrayPool<byte>.Shared.Return(buffer); 
}
```

Bu desen, GC üzerindeki baskıyı dramatik şekilde azaltır. Ancak kiralanan dizinin "temiz" olmayabileceği (önceki kullanımdan veri içerebileceği) ve istenenden daha büyük olabileceği unutulmamalıdır.54

### 10.3 `stackalloc`: Güvenli Yığın Tahsisi

`stackalloc` anahtar kelimesi, dizinin Heap yerine Stack üzerinde tahsis edilmesini sağlar. Stack tahsisi nanosaniyeler mertebesindedir ve GC'ye hiç yük bindirmez. Modern C# ile `stackalloc`, `Span<T>` ile birleşerek `unsafe` bloğuna ihtiyaç duymadan güvenli bir şekilde kullanılabilir.

C#

```
Span<int> numbers = stackalloc int;
```

Bu yöntem, sadece çok küçük ve metodun yaşam süresiyle sınırlı diziler için kullanılmalıdır. Büyük tahsisler `StackOverflowException` riskini doğurur.50

## Sonuç

.NET Core ekosisteminde veri yapıları ve algoritmalar, teorik bilgisayar bilimleri ile pratik sistem mühendisliğinin kesişim noktasındadır. Bir algoritmanın Big O karmaşıklığı önemli bir başlangıç noktası olsa da,.NET dünyasında "gerçek" performans; bellek yerelliği, tahsis maliyetleri, GC davranışı ve doğru veri yapısı seçimi (örneğin `struct` vs `class`, `List` vs `LinkedList`) ile belirlenir. Modern.NET araçları (`Span`, `ArrayPool`, `Introsort`, `PriorityQueue`), geliştiricilere hem yüksek seviyeli soyutlamalar hem de düşük seviyeli bellek kontrolü sunarak, optimize edilmiş yazılımlar geliştirme imkanı tanımaktadır. Bu yetkinlikler, sıradan bir kod yazarı ile bir performans mimarı arasındaki farkı belirleyen temel unsurlardır.

### Tablo Listesi

- **Tablo 1:** Zaman Karmaşıklığı Sınıfları ve.NET Örnekleri
    
- **Tablo 2:** `SortedDictionary` ve `SortedList` Karşılaştırması
    

---

**Atıflar:** 2