Bu 10 madde, veritabanı dünyasının "Anayasası"dır. Bir Backend Developer olarak bu maddelerin bazıları doğrudan seni ilgilendirir (Kodlama), bazıları ise DevOps veya DBA (Database Admin) ekiplerini ilgilendirir (Altyapı). Ancak hepsini bilmek, seni "Junior" seviyesinden "Güvenlik Bilinci Olan Mühendis" seviyesine taşır.

Güvenlik tek bir duvar değildir; soğan gibi katmanlardan oluşur (**Defense in Depth**).

Senin için en kritik olan maddeleri bir Developer gözüyle detaylandıralım:

---

### Backend Developer İçin Hayati Maddeler

Bu listedeki 3 madde, doğrudan senin yazdığın C# veya Python koduyla ilgilidir. Mülakatlarda bunları vurgulaman çok puan kazandırır.

#### 1. SQL Injection (Madde 10) - _En Büyük Düşman_

Listendeki en önemli madde budur. Çünkü dünyanın en güvenli sunucusunu da kursan, kodunda SQL Injection açığı varsa, 12 yaşındaki bir çocuk bile veritabanını ele geçirebilir.

- **Hatalı (String Birleştirme):** `var sql = "SELECT * FROM Users WHERE Name = '" + userName + "'";`
    
    - Kullanıcı ismine `' OR '1'='1` yazarsa, sorgu `SELECT * FROM Users` haline gelir ve tüm şifreler dökülür.
        
- **Doğru (Parameterized Query):** `var sql = "SELECT * FROM Users WHERE Name = @Name";`
    
    - C# veya Python, gönderilen veriyi "Komut" olarak değil, sadece "Metin" olarak işler. Hacklemek imkansız hale gelir.
        

#### 2. Least Privilege & Admin Account (Madde 1 ve 5)

`appsettings.json` veya `.env` dosyasına yazdığın Connection String'de **asla** `sa` (System Admin) veya `root` kullanma.

- **Senaryo:** Uygulaman hacklendi.
    
- **Admin İle Bağlandıysan:** Hacker `DROP DATABASE` yazar, şirketi batırır.
    
- **Kısıtlı Kullanıcı İle Bağlandıysan:** Hacker en fazla o tablodan veri okur. Hasarı minimize edersin (Damage Control).
    

#### 3. Encrypt Communication (Madde 6)

Veritabanı sunucusu ile Backend sunucusu farklı makinelerdeyse, aradaki trafik şifrelenmelidir (SSL/TLS).

- **Neden?** Aynı ağdaki kötü niyetli biri (Man-in-the-Middle), ağ trafiğini dinleyerek (Sniffing) veritabanına giden şifreleri veya müşteri verilerini düz metin olarak yakalayabilir. Connection String'de `Encrypt=True` parametresini kullanmak bu yüzden önemlidir.
    

---

### Altyapı ve Bakım Maddeleri (Kısa Notlar)

Diğer maddeler genellikle veritabanı yöneticisinin sorumluluğundadır ama senin de bilmen gerekir:

- **Regular Updates (Madde 2):** "WannaCry" gibi virüsler, güncellenmemiş sistemleri sever. SQL Server'ı güncel tutmak, bilinen arka kapıları kapatır.
    
- **Database Backups (Madde 7):** Yedek, paraşüt gibidir. İhtiyacın olduğunda yoksa, çakılırsın. "3-2-1 Kuralı"nı unutma (3 kopya, 2 farklı medya, 1'i bina dışında).
    
- **Monitoring (Madde 8):** Logları izlemek. "Gece saat 03:00'te kim `Users` tablosundan 10.000 kayıt çekti?" sorusunun cevabı buradadır.
    
- **Vulnerability Scanning (Madde 9):** Sistemin açıklarını otomatik tarayan araçlar kullanmak (Örn: Nessus, SQLMap).