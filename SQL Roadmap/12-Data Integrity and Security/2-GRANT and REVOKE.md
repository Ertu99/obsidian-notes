Bu komutlar, veritabanının "Erişim Kontrol Sistemi"dir (Access Control). Constraints ile verinin **doğruluğunu** korumuştuk, bu komutlarla verinin **gizliliğini** ve **güvenliğini** koruruz.

**Temel Felsefe: "Least Privilege" (En Az Yetki Prensibi)** Bir güvenlik görevlisi binadaki her kapıyı açan anahtarı temizlikçiye vermez. Sadece temizleyeceği odanın anahtarını verir. Veritabanında da kural budur: Bir kullanıcıya (veya uygulamaya) sadece **işini yapmasına yetecek kadar** izin verilir, fazlası değil.

---

### 1. GRANT (İzin Verme)

Bir kullanıcıya veya bir role belirli bir işlemi yapma hakkı tanır.

- **Senaryo:** "Stajyer Ahmet sadece `Siparisler` tablosunu okuyabilsin (`SELECT`), ama silemesin veya yeni sipariş giremesin."
    
- **Syntax:** `GRANT [YETKİ] ON [TABLO] TO [KULLANICI]`
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Ahmet sadece okuyabilir
    GRANT SELECT ON Orders TO Ahmet;
    
    -- Muhasebe Müdürü hem okuyabilir hem güncelleyebilir
    GRANT SELECT, UPDATE ON Orders TO MuhasebeMuduru;
    ```
    

### 2. REVOKE (İzni Geri Alma)

Daha önce `GRANT` ile verilmiş bir yetkiyi iptal eder.

- **Senaryo:** "Stajyer Ahmet işten ayrıldı" veya "Artık o projede çalışmıyor."
    
- **Syntax:** `REVOKE [YETKİ] ON [TABLO] FROM [KULLANICI]`
    
- **Örnek:**
    
    SQL
    
    ```sql
    -- Ahmet'in okuma yetkisini elinden al
    REVOKE SELECT ON Orders FROM Ahmet;
    ```
    
- **Önemli Not:** `REVOKE`, bir yasağı (`DENY`) başlatmaz; sadece verilmiş bir izni geri alır. Eğer Ahmet bir "Yönetici Grubu"na üyeyse ve o grubun izni varsa, Ahmet gruptan dolayı hala erişebilir.
    

---

### Best Practice: Roller (Roles) ile Yönetim

Binlerce çalışanın olduğu bir şirkette herkese tek tek `GRANT` yazmak deliliktir. Bunun yerine **Roller (Gruplar)** kullanılır.

1. **Rol Oluştur:** `CREATE ROLE SatisEkibi`
    
2. **Role Yetki Ver:** `GRANT INSERT, SELECT ON Orders TO SatisEkibi` (Satış ekibi sipariş girebilir ve görebilir).
    
3. **Kullanıcıyı Role Ekle:** `ALTER ROLE SatisEkibi ADD MEMBER Ahmet`
    
4. **Ahmet Gidince:** Sadece rolden çıkarırsın. Tek tek `REVOKE` ile uğraşmazsın.
    

---

### Backend Developer İçin Neden Önemli?

1. **Connection String Güvenliği:** C# projesinde `appsettings.json` dosyasına yazdığın veritabanı kullanıcısı (User Id / Password) asla **`sa` (System Admin)** olmamalıdır!
    
    - **Neden?** Eğer uygulaman hacklenirse (SQL Injection vb.), saldırgan `sa` yetkisiyle tüm veritabanını silebilir (`DROP DATABASE`).
        
    - **Doğrusu:** Uygulama için özel bir kullanıcı (örn: `AppUser`) oluşturulmalı ve sadece `GRANT SELECT, INSERT, UPDATE` yetkileri verilmelidir. `DROP` veya `TRUNCATE` yetkisi **asla** verilmemelidir.
        
2. **ReadOnly Veritabanı Bağlantıları:** Büyük projelerde raporlama ekranları için ayrı bir `ReadOnlyUser` oluşturulur. Bu kullanıcıya sadece `GRANT SELECT` verilir. Backend tarafında rapor çekerken bu connection string kullanılır. Böylece yanlışlıkla veri silme riski sıfıra iner.