
Şimdi Unit Test dünyasının en büyük "Sihirbazı"na, **Moq** kütüphanesine geliyoruz.

Unit Test'in tanımı şuydu: "İzolasyon". Yani ProductService test edilirken, veritabanına, ağa veya disk sistemine gidilmemeli.

Peki ProductService constructor'ında IProductRepository istiyorsa ne yapacağız?

Ona "Yalan Söyleyen Nesne" (Mock) vereceğiz.

Bu konuyu; **Stub vs Mock** farkı, **Setup & Verify** mekaniği ve yakın zamanda yaşanan **Güvenlik Skandalı (SponsorLink)** üzerinden mühendislik derinliğinde inceleyelim.

---

### 1. Felsefe: Stub vs Mock (Kavram Kargaşası)

Herkes "Mock" kelimesini kullanır ama Martin Fowler bunları ayırır:

1. **Stub (Taslak):** Durum (State) odaklıdır.
    
    - _"Repository'ye 'Get(5)' sorulursa, şu User nesnesini dön."_
        
    - Amacımız servisin dönüş değerini test etmektir.
        
2. **Mock (Taklit):** Davranış (Behavior) odaklıdır.
    
    - _"Servis çalışırken Repository'nin 'Delete' metodu tam olarak 1 kere çağrıldı mı?"_
        
    - Amacımız yan etkileri (Side Effect) doğrulamaktır.
        

Moq kütüphanesi bu ikisini de yapabilir.

---

### 2. Temel Mekanik: `Setup` ve `Object`

Moq, Reflection kullanarak çalışma zamanında (Runtime) senin Interface'inden (`IUserRepository`) türeyen sahte bir sınıf (Proxy) oluşturur.

C#

```cs
// 1. Mock Nesnesini Yarat (Wrapper)
var mockRepo = new Mock<IUserRepository>();

// 2. Senaryoyu Kur (Stubbing)
// "GetUser(1) çağrılırsa, Ahmet'i dön."
mockRepo.Setup(x => x.GetUser(1))
        .Returns(new User { Id = 1, Name = "Ahmet" });

// 3. Test Edilen Sınıfa Enjekte Et
// DİKKAT: mockRepo'yu değil, mockRepo.Object'i veriyoruz!
var service = new UserService(mockRepo.Object);
```

---

### 3. Esnek Parametreler: `It.IsAny<T>`

Test yazarken bazen parametrenin tam değeriyle ilgilenmezsin.

"Kullanıcı hangi ID ile çağrılırsa çağrılsın, sonuç dönsün" demek isteyebilirsin.

C#

```cs
// ID 1 de gelse, 999 da gelse bu user döner.
mockRepo.Setup(x => x.GetUser(It.IsAny<int>()))
        .Returns(new User { Name = "Genel Kullanıcı" });

// string parametre "Test" ile başlıyorsa kabul et
mockRepo.Setup(x => x.FindUser(It.Is<string>(s => s.StartsWith("Test"))))
        .Returns(new User());
```

---

### 4. Davranış Doğrulama: `Verify`

Burası Mocking'in gerçek gücüdür. Diyelim ki DeleteUser metodunu test ediyorsun. Bu metot geriye bir şey dönmüyor (void). Testin başarılı olduğunu nasıl anlarsın?

Cevap: Repository'nin silme komutunu alıp almadığını kontrol ederek.

C#

```cs
// Act
service.DeleteUser(5);

// Assert (Doğrulama)
// "Delete metodu, ID=5 parametresiyle, TAM OLARAK 1 KERE çağrıldı mı?"
mockRepo.Verify(x => x.Delete(5), Times.Once);
```

Eğer kodunda bir hata varsa ve `Delete` metodu hiç çağrılmadıysa veya 2 kere çağrıldıysa test patlar.

---

### 5. Mutsuz Yol: Exception Fırlatmak

Unit Test sadece "Mutlu Yol"u (Happy Path) değil, hataları da test etmelidir.

"Veritabanı çökerse servisim ne yapıyor?"

C#

```cs
// "Save çağrıldığında veritabanı hatası fırlat"
mockRepo.Setup(x => x.Save(It.IsAny<User>()))
        .Throws(new SqlException(...));

// Sonra servisin bu hatayı düzgün yakaladığını (try-catch) test et.
```

---

### 6. Loose vs Strict Mock (Gevşek vs Katı)

Moq varsayılan olarak **Loose** modda çalışır.

- **Loose:** Eğer `Setup` yapmadığın bir metodu çağırırsan, hata vermez. Varsayılan değeri (null veya 0) döner. Testler daha az kırılgandır.
    
- **Strict:** `Setup` yapılmamış bir metot çağrılırsa anında `MockException` fırlatır. _"Benim iznim olmadan hiçbir şeye dokunamazsın"_ der. Çok sıkı kontrol gereken yerlerde kullanılır.
    

C#

```cs
var mock = new Mock<IUserRepository>(MockBehavior.Strict);
```

---

### 7. Güvenlik Skandalı: SponsorLink Olayı (Architect Note)

Bir Mimar olarak açık kaynak kütüphanelerin risklerini bilmelisin.

2023 yılında Moq'un geliştiricisi, kütüphanenin içine SponsorLink adında kapalı kaynak bir kod ekledi. Bu kod, testi çalıştıran geliştiricinin e-posta adresini (hashlenmiş olarak) bir sunucuya gönderiyordu.

Bu olay dünya çapında büyük tepki çekti (GDPR ihlali, Güven eksikliği). Microsoft ve pek çok şirket Moq kullanımını yasaklayıp NSubstitute kütüphanesine geçti.

Sonradan bu özellik geri çekildi, ancak güven bir kere sarsıldı.

Ders: Kullandığın kütüphanenin versiyon güncellemelerini ve topluluk tepkilerini takip etmelisin.

---
