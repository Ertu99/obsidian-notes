Burası veritabanı mülakatlarının **"Boss Level"** sorusudur. İzolasyon seviyeleri, ACID prensibindeki "I" (Isolation) maddesinin ayarlarıdır.

**Mantık:** "Bir Transaction işlem yaparken, diğer Transaction'lar onu ne kadar rahatsız edebilir?" sorusunun cevabıdır.

- **Düşük Seviye:** Performans uçar ama veriler tutarsız olabilir.
    
- **Yüksek Seviye:** Veri beton gibi sağlamdır ama sistem yavaşlar ve kilitlenmeler (Deadlock) artar.
    

Bu seviyeleri anlamak için önce çözdükleri 3 temel sorunu (The 3 Villains) tanıman lazım:

1. **Dirty Read (Kirli Okuma):** Başkasının henüz `COMMIT` etmediği (belki de vazgeçeceği) veriyi okumak.
    
2. **Non-Repeatable Read (Tekrarlanamayan Okuma):** Aynı Transaction içinde bir satırı iki kere okuduğunda, arada başkası `UPDATE` ettiği için farklı değer gelmesi.
    
3. **Phantom Read (Hayalet Okuma):** Aynı sorguyu iki kere çalıştırdığında, arada başkası `INSERT` yaptığı için satır sayısının değişmesi (Örn: İlkinde 10 kayıt vardı, ikincisinde 11 oldu).
    

Şimdi bu sorunlara göre 4 seviyeyi inceleyelim:

---

### 1. Read Uncommitted (En Gevşek Seviye)

**Motto:** "Her şeyi görürüm, gizlilik yok." En tehlikeli ama en hızlı seviyedir. Başka bir Transaction'ın henüz kaydetmediği (uncommitted) veriyi okuyabilir.

- **Risk:** **Dirty Read** oluşur. Ahmet hesabına para yatırır (henüz onaylamaz), sen bakiyeyi artmış görürsün. Sonra Ahmet işlemi iptal eder (`ROLLBACK`), senin elinde olmayan bir bakiye bilgisi kalır.
    
- **Kullanım:** Sadece %100 doğruluk gerektirmeyen anlık raporlarda (Örn: "Şu an sitede kaç kişi var?") kullanılır. `NOLOCK` olarak da bilinir.
    

### 2. Read Committed (Varsayılan Standart)

**Motto:** "Sadece imzalanmış evrakları okurum." SQL Server, PostgreSQL ve Oracle'ın varsayılan (Default) ayarıdır. Bir veri okunabilmesi için mutlaka `COMMIT` edilmiş olmalıdır.

- **Çözüm:** Dirty Read engellenir.
    
- **Risk:** **Non-Repeatable Read** olabilir. Sen rapor çekerken Ali fiyatı değiştirip Commit ederse, raporun başında fiyat 10 TL, sonunda 15 TL görünebilir.
    

### 3. Repeatable Read (Tutarlı Okuma)

**Motto:** "Ben okurken kimse dokunamaz." Okuduğun satırlara kilit (Lock) koyar. Sen işlemini bitirene kadar kimse o satırları güncelleyemez (`UPDATE` yapamaz).

- **Çözüm:** Dirty Read ve Non-Repeatable Read engellenir.
    
- **Risk:** **Phantom Read** olabilir. Sen "Ankara'daki Müşteriler"i listelerken kilit koydun. Kimse mevcut Ankaralıları güncelleyemez ama araya yeni bir Ankaralı müşteri (`INSERT`) eklenebilir. Tekrar sorgularsan "Hayalet" bir kayıt belirir.
    

### 4. Serializable (En Sıkı Seviye)

**Motto:** "Mekanı kapattım, içeride sadece ben varım." En güvenli ama en yavaş seviyedir. Sanki Transactionlar sıraya girmiş gibi tek tek çalışır.

- **Çözüm:** Tüm sorunlar (Dirty, Non-Repeatable, Phantom) engellenir. Range Lock (Aralık Kilidi) kullanır.
    
- **Bedel:** Performans dibe vurur. Deadlock (Ölümcül Kilitlenme) ihtimali çok yüksektir. Sadece finansal hesaplamalar gibi hatanın kabul edilemeyeceği yerlerde kullanılır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Deadlock Analizi:** Uygulaman sürekli "Deadlock victim" hatası veriyorsa, muhtemelen gereksiz yere yüksek izolasyon seviyesi (Serializable gibi) kullanıyorsundur veya transaction sürelerin çok uzundur.
    
2. **`WITH(NOLOCK)` Kültürü:** Büyük sistemlerde rapor çekerken, canlı sistemi kilitlememek için sorgunun sonuna `WITH(NOLOCK)` eklenir. Bu, o sorgu için izolasyon seviyesini "Read Uncommitted"a çeker.
    
    - _Örnek:_ `SELECT * FROM Orders WITH(NOLOCK)`
        
3. **Entity Framework Ayarı:** EF Core kullanırken Transaction başlattığında izolasyon seviyesini seçebilirsin:
    
    C#
    
    ```csharp
    using (var transaction = context.Database.BeginTransaction(IsolationLevel.Serializable))
    {
        // Çok kritik işlem
    }
    ```
    
    Bunu belirtmezsen veritabanının varsayılan ayarı (genelde Read Committed) geçerli olur.