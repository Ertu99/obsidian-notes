
Şimdi Unit Test dünyasının kralı, modern .NET ekosisteminin varsayılan standardı olan **xUnit**'e geçiyoruz.

MSTest ve NUnit "eski kafa" (Legacy) yaklaşımlara sahiptir. xUnit ise; testleri birbirinden izole etmek, temiz kod yazmaya zorlamak ve modern (Data-Driven) testler yazmak için sıfırdan tasarlanmıştır.

Bu konuyu; **Lifecycle (Yaşam Döngüsü)**, **Data-Driven Tests (Theory)** ve **Shared Context (Fixture)** mimarisi üzerinden eksiksiz inceleyelim.

---

### 1. Felsefe: Neden `[SetUp]` ve `[TearDown]` Yok?

MSTest veya NUnit kullananlar şuna alışıktır: [SetUp] metodu her testten önce çalışır.

xUnit geliştiricileri buna karşı çıkar: "Test sınıfı, normal bir C# sınıfı gibi davranmalıdır."

- **Setup Yerine:** **Constructor (Yapıcı Metot)** kullanılır.
    
- **TearDown Yerine:** **`IDisposable.Dispose`** metodu kullanılır.
    

**Mühendislik Kuralı:** xUnit, **her test metodu için sınıfın yeni bir örneğini (Instance) oluşturur.**

- Sınıfta 5 test metodu (`[Fact]`) varsa, Constructor 5 kere çalışır.
    
- Bu, **Test İzolasyonu** (Isolation) sağlar. Test A'nın değiştirdiği bir değişken, Test B'yi etkilemez.
    

---

### 2. Fact vs Theory (Veri Odaklı Testler)

Unit test yazarken en büyük amelelik, aynı senaryoyu farklı verilerle test etmektir.

- "Yaş 17 ise Reddet."
    
- "Yaş 18 ise Onayla."
    
- "Yaş 90 ise Onayla."
    

Bunun için 3 ayrı metot yazmak yerine **`[Theory]`** kullanırız.

C#

```cs
[Theory]
[InlineData(17, false)]
[InlineData(18, true)]
[InlineData(90, true)]
public void IsAdult_ShouldReturnExpectedResult(int age, bool expected)
{
    var result = _service.IsAdult(age);
    Assert.Equal(expected, result);
}
```

**Güçlü Yanı:** Eğer testin verisi sabit değilse (örneğin karmaşık nesnelerse), `[MemberData]` veya `[ClassData]` kullanarak veriyi başka bir C# metodundan dinamik olarak besleyebilirsin.


### 3. Shared Context (Fixture Mimarisi)

Her test için Constructor çalışır dedik. Peki ya "Veritabanı Bağlantısı" gibi kurulması çok pahalı bir nesnen varsa? Her testte veritabanına bağlanıp koparsan testlerin dakikalar sürer.

xUnit buna **Fixture** çözümü sunar:

1. **IClassFixture<T>:** Bir test sınıfındaki **tüm metotlar** arasında tek bir nesneyi paylaşır.
    
    - Sınıf yaratılırken 1 kere oluşur, tüm testler bitince 1 kere yok edilir.
        
2. **ICollectionFixture<T>:** **Farklı test sınıfları** arasında aynı nesneyi paylaşır.
    
    - "UserTests" ve "OrderTests" sınıflarının ikisi de aynı Docker konteynerini kullansın istersen bunu kullanırsın.
        

**Örnek (ClassFixture):**

C#

```
public class DatabaseTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public DatabaseTests(DatabaseFixture fixture) // Singleton gibi gelir
    {
        _fixture = fixture;
    }
}
```

---

### 4. Assertions vs FluentAssertions

xUnit'in kendi `Assert` sınıfı yeterlidir (`Assert.Equal(5, result)`). Ancak profesyonel dünyada biz **FluentAssertions** kütüphanesini kullanırız.

- **xUnit:** `Assert.Equal(expected, actual);` (Bazen hangisi expected hangisi actual karışır).
    
- **Fluent:** `actual.Should().Be(expected);`
    
    - `result.Should().StartWith("Err").And.Contain("Null");`
        
    - Okunabilirliği muazzam artırır ve hata mesajları çok daha açıklayıcıdır.
        

---

### 5. Parallel Execution (Hız ve Tehlike)

xUnit varsayılan olarak **Test Collection** bazında paralel çalışır.

- `UserTests.cs` ve `ProductTests.cs` dosyaları **aynı anda** (farklı threadlerde) çalıştırılır.
    
- Bu sayede test süresi kısalır.
    

**Mühendislik Riski:** Eğer bu iki sınıf, ortak bir statik kaynağı (örneğin statik bir Listeyi veya aynı veritabanı tablosunu) değiştiriyorsa **Race Condition** oluşur ve testler rastgele patlar.

- **Çözüm:** Paylaşılan kaynak kullanan testleri aynı `[Collection]` içine koyarak "Sırayla çalışın" emri vermek.


**🧒 6 Yaşındaki Çocuğa (Lego Seti Analojisi):** "Lego oynarken her seferinde yeni bir kale yapmak istediğini düşün. Eski oyunlarda (**MSTest/NUnit**), bir kaleyi bitirince onu tam bozmazdın, yarısını bırakıp üzerine yeni kale yapardın. Ama bazen eski kalenin parçaları yeni kaleyi çirkin yapardı (**State Leakage**). **xUnit** ise çok titiz bir oyun kurucudur. Her yeni kale yapacağında, yerdeki her şeyi tamamen süpürür ve kutuyu sıfırdan açar (**Constructor**). Böylece önceki oyunun sonraki oyununu bozamaz. Ama her seferinde sıfırdan yapmak çok uzun sürüyorsa, mesela kocaman bir Lego masasına ihtiyacın varsa; masayı bir kere kurarsın, bütün oyunlarını o masanın üzerinde oynarsın, işin bitince masayı kaldırırsın (**ClassFixture**). Bir de bazen talimat kitapçığında 'Bu parçayı tak' yazar. Kırmızı parça için de, mavi parça için de aynı hareketi yaparsın. xUnit buna da izin verir; tek kural yazarsın, farklı renkleri denersin (**Theory**)."

**👨‍💼 Mülakatta Yöneticiye (Abstraction - Teorik Uzman Dili):** ".NET Core ekosisteminde Unit Test standartları, test izolasyonunu ve güvenilirliğini (Reliability) en üst düzeye çıkaran **xUnit** üzerine kuruludur. Mimari yaklaşım, testlerin birbirini etkilememesi (State Isolation) prensibine dayanır:

- **Lifecycle Management:** Geleneksel `[SetUp]` ve `[TearDown]` metodları yerine; xUnit'in her test metodu için sınıfı yeniden örneklendirmesi (Constructor) ve temizlik için `IDisposable` arayüzünü kullanması, 'paylaşılan durum' (Shared State) hatalarını mimari seviyede engeller.
    
- **Context Efficiency:** Veritabanı bağlantısı veya Docker container gibi oluşturulması maliyetli kaynaklar için, her testte sıfırdan kurulum yapmak yerine **Fixture (`IClassFixture`)** deseni kullanılarak bu kaynakların yaşam döngüsü optimize edilir.
    
- **Code Quality & Coverage:** İş mantığını farklı veri setleriyle doğrulamak için kod tekrarı yerine **Data-Driven Test (`[Theory]`)** yetenekleri kullanılır. Ayrıca test kodlarının okunabilirliğini artırmak ve 'Dokümantasyon' niteliği kazandırmak için **FluentAssertions** kütüphanesi standart olarak projeye dahil edilir.
    
- **Performance:** xUnit'in varsayılan olarak sunduğu **Paralel Çalıştırma** (Parallel Execution) yeteneği, CI/CD süreçlerini hızlandırır; ancak bu güç, 'Race Condition' risklerine karşı `Collection` tanımlamalarıyla kontrollü yönetilmelidir."