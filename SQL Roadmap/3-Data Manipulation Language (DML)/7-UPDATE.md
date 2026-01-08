**UPDATE**

`UPDATE`, veritabanında daha önce kaydedilmiş bir veriyi **değiştirmek** için kullanılan DML komutudur.

**Neden Kullanılır?** Kullanıcı şifresini unuttu ve değiştirmek istedi, e-ticaret sitesinde ürünün fiyatına zam geldi veya kargo durumu "Hazırlanıyor"dan "Yola Çıktı"ya döndü. Tüm bu yaşayan süreçler `UPDATE` komutuyla yönetilir. Veritabanı sadece bir arşiv değil, dinamik bir yapıdır.

---

### Deep Dive: Değişimin Riskleri ve Teknik Detaylar

Bir Junior Developer olarak `SELECT` sorgusunda hata yaparsan en fazla yanlış veri gelir. Ancak `UPDATE` sorgusunda hata yaparsan **veri bozulur**. Bu yüzden en tehlikeli komutlardan biridir.

#### 1. Unutulan WHERE Faciası - _En Büyük Kâbus_

Mülakatlarda veya iş hayatında kulağına küpe olması gereken kural:

- **Hatalı:** `UPDATE Products SET Price = 100`
    
    - **Sonuç:** `WHERE` koşulu yazmadığın için veritabanındaki **TÜM** ürünlerin fiyatı 100 TL olur. Geri dönüşü yoktur (Backup yoksa).
        
- **Güvenli:** `UPDATE Products SET Price = 100 WHERE Id = 502`
    
    - Sadece ilgili satırı günceller.
        
- **Taktik:** Manual sorgu atarken önce `SELECT * FROM ... WHERE ...` ile doğru veriyi seçtiğinden emin ol, sonra o `WHERE` bloğunu `UPDATE` sorgusuna kopyala.
    

#### 2. Başka Tabloya Göre Güncelleme (UPDATE with JOIN)

Bazen güncelleme kriterin o tabloda değil, başka bir tablodadır.

- **Senaryo:** "Sadece 'Elektronik' kategorisindeki ürünlere %10 zam yap." (Kategori bilgisi `Categories` tablosunda, Fiyat `Products` tablosunda).
    
- **Yöntem:**
    
    SQL
    
    ```sql
    UPDATE p
    SET p.Price = p.Price * 1.10
    FROM Products p
    INNER JOIN Categories c ON p.CategoryId = c.Id
    WHERE c.Name = 'Elektronik'
    ```
    
    Bu yapı (JOIN ile UPDATE), SQL mülakatlarında "Advanced" kategorisinde sorulur.
    

#### 3. Transaction ve Locking (Kilitleme)

`UPDATE` işlemi, veriyi değiştirirken o satırı (bazen sayfayı veya tabloyu) **kilitler (Lock)**.

- Sen bir ürünü güncellerken (Transaction bitene kadar), başka bir müşteri o ürünü görüntüleyemez veya satın alamaz (Bekler).
    
- Eğer çok büyük bir `UPDATE` (örneğin 1 milyon satır) yaparsan, sistem kilitlenir. Bu yüzden toplu güncellemeler parça parça (Batch) yapılır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Entity Framework Core Change Tracking:** .NET'te kod yazarken genelde SQL `UPDATE` yazmazsın.
    
    - Önce veriyi çekersin: `var user = context.Users.Find(1);`
        
    - Değişikliği yaparsın: `user.Name = "Mehmet";`
        
    - Kaydedersin: `context.SaveChanges();`
        
    - EF Core, bu nesnenin değiştiğini anlar (Change Tracker) ve arka planda sadece değişen alan için `UPDATE Users SET Name = 'Mehmet' WHERE Id=1` kodunu üretir.
        
2. **Concurrency (Eşzamanlılık) - Optimistic Locking:** Aynı anda iki admin bir ürünün fiyatını güncellemeye çalışırsa ne olur?
    
    - Backend Developer olarak SQL tarafında `RowVersion` veya `Timestamp` kullanırız.
        
    - `UPDATE ... WHERE Id = 1 AND Version = 5`
        
    - Eğer işlem sırasında versiyon değişmişse (başkası güncellemişse), senin işlemin başarısız olur (`0 rows affected`). Buna **Concurrency Control** denir ve mülakatların kilit sorusudur.
        
3. **Soft Delete (Mantıksal Silme):** Modern sistemlerde veriler `DELETE` ile silinmez. Bunun yerine `UPDATE` kullanılır.
    
    - `UPDATE Users SET IsDeleted = 1 WHERE Id = 5`
        
    - Kullanıcı silinmiş gibi görünür ama veri tabanında durur. Backend kodların bunu yönetmelidir.