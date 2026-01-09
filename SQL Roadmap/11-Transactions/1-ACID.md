ACID, İlişkisel Veritabanı Sistemlerinin (RDBMS) "Güvenilirlik" sözleşmesidir. Bir sistemin "Ben finansal işlemler için güvenliyim" diyebilmesi için bu 4 maddeye kefil olması gerekir.

Mülakatlarda en sık sorulan bu kavramları, bir Backend Developer olarak ezbere değil, mantığıyla bilmelisin. Gel detaylarına inelim:

---

### 1. **A**tomicity (Atomiklik / Bölünemezlik)

**Motto:** "Ya Hep Ya Hiç" (All or Nothing). Bir Transaction içindeki işlemler atom gibidir, parçalanamaz. 10 adımdan oluşan bir işlem zincirin varsa, 9'u başarılı olup 10. adım başarısız olursa, sistem ilk 9 adımı da geri alır (**Rollback**). Hiçbir şey olmamış gibi davranır.

- **Örnek:** ATM'den para çekerken hesabından para düştü ama tam para verilecekken elektrik kesildi.
    
    - **Atomicity Yoksa:** Paran hesabından gitti ama cebine girmedi.
        
    - **Atomicity Varsa:** Sistem paranın verilmediğini anlar ve hesaptan düşülen parayı anında iade eder.
        

### 2. **C**onsistency (Tutarlılık)

**Motto:** "Kurallar Çiğnenemez." Transaction başlamadan önce veritabanı kurallara (Constraints) uygunsa, bittiğinde de uygun olmalıdır. Veritabanının bütünlüğü bozulamaz.

- **Örnek:** Veritabanında "TC Kimlik No boş olamaz" (`NOT NULL`) veya "Bakiye eksiye düşemez" (`CHECK constraint`) kuralları var.
    
- Sen bir Transaction içinde bakiyeyi eksiye düşürecek bir işlem yapmaya kalkarsan, Consistency kuralı devreye girer ve işlemi iptal eder. Veritabanını bozuk (invalid) bir durumda bırakmaz.
    

### 3. **I**solation (İzolasyon / Yalıtım)

**Motto:** "Başkası ne yapıyor beni ilgilendirmez." Aynı anda binlerce kullanıcı işlem yapıyor olabilir (Concurrency). Her işlem sanki dünyada tek başına çalışıyormuş gibi diğerlerinden yalıtılmalıdır. Bir işlem bitmeden (Commit olmadan), diğer kullanıcılar o işlemin ara adımlarını göremez.

- **Örnek:** Son kalan konser biletini aynı anda Ali ve Veli satın almaya çalışıyor.
    
- **Isolation Varsa:** Sistem bu iki isteği sıraya koyar (Locking). Ali işlemi bitirene kadar Veli bekler (veya hata alır). İkisi aynı anda aynı koltuğu alamaz.
    

### 4. **D**urability (Dayanıklılık / Kalıcılık)

**Motto:** "Söz uçar, yazı kalır." Sistem sana "İşlem Başarılı" (Commit) cevabını döndüğü milisaniyede, o veri artık diskte (Hard Drive/SSD) garanti altındadır.

- **Senaryo:** Ekranda "Kayıt Başarılı" yazısını gördün ve tam 1 saniye sonra sunucu odasında yangın çıktı, fişler çekildi.
    
- **Durability:** Sistem tekrar ayağa kalktığında o verinin orada olacağını garanti eder. Bunu sağlamak için veritabanları "Transaction Log" (Write-Ahead Logging) denilen mekanizmayı kullanır. Veri tabloya yazılmadan önce günlüğe yazılır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Race Conditions (Yarış Durumu):** Yazılımcıların en büyük kâbusu "Ben test ederken çalışıyordu, canlıda veriler karıştı" durumudur. Bu genellikle **Isolation** eksikliğinden kaynaklanır. C# veya Python kodunda her şey doğru olsa bile, veritabanı seviyesinde izolasyonu doğru ayarlamazsan, aynı anda işlem yapan kullanıcılar birbirinin verisini ezer.
    
2. **Veri Tutarlılığı:** Eğer bir E-Ticaret sitesi yazıyorsan ve `Siparisler` tablosuna kayıt atıp `Stok` tablosundan düşmeyi unutursan (veya hata alırsan), ACID prensipleri sayesinde sistemi geri alıp stoğun yanlış görünmesini engellersin.
    
3. **Distributed Transactions (Dağıtık Sistemler):** Microservices dünyasında ACID sağlamak zordur (çünkü veriler farklı veritabanlarında olabilir). Burada "Saga Pattern" veya "Two-Phase Commit" gibi ileri seviye konular devreye girer ama temeli yine ACID'i simüle etmeye dayanır.
    

ACID, ilişkisel veritabanlarını (SQL Server, PostgreSQL, MySQL) NoSQL'den ayıran en büyük güçtür.