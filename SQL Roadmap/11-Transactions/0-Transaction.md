SQL dünyasının en hayati konularından birine giriş yapıyoruz. Transaction, "Ya Hep Ya Hiç" (All or Nothing) felsefesinin teknik adıdır. Birden fazla SQL sorgusunu tek bir paket (Unit of Work) haline getirir.

**Neden Kullanılır?** En klasik örnek banka havalesidir. Ahmet'in hesabından 100 TL düşüp, Mehmet'in hesabına 100 TL ekleyeceksin.

1. Ahmet'ten 100 TL düş. (Başarılı)
    
2. Elektrikler kesildi veya sistem hata verdi.
    
3. Mehmet'e 100 TL ekle. (Çalışmadı)
    

Eğer Transaction kullanmazsan; Ahmet'in parası gitti ama Mehmet'e gelmedi. Para buhar oldu. Transaction kullanırsan, 2. adımda hata olduğu an sistem **her şeyi iptal eder** ve Ahmet'in parasını geri koyar.

### Backend Developer İçin Neden Önemli?

1. **Veri Bütünlüğü (Data Integrity):** E-ticaret sitesinde "Sipariş Oluştur" butonuna basıldığında arkada 3 iş döner:
    
    - Stoktan düş.
        
    - Kredi kartından para çek.
        
    - Sipariş kaydı oluştur.
        
    - Bu işlemlerden biri patlarsa (örneğin kartta limit yoksa), stoktan düşülen ürünün geri gelmesi gerekir. Bunu kodla `if-else` yazarak yönetmek zordur, Transaction ile yönetmek tek satırdır.
        
2. **Hata Yönetimi:** Python veya C# tarafında `try-catch` bloğu içinde veritabanı işlemleri yaparken, `catch` bloğunda `transaction.Rollback()` diyerek veritabanını tertemiz bırakabilirsin.
    

Transaction kavramı, veritabanının "Güvenilir Liman" olmasını sağlayan temel yapıdır.