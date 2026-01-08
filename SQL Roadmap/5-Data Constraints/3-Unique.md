**UNIQUE (Benzersizlik)**

`UNIQUE` kısıtlaması, bir sütundaki (veya sütun grubundaki) verilerin **tekrar etmemesini**, yani her satırda birbirinden farklı olmasını garanti altına alan kuraldır.

**Neden Kullanılır?** Bir kullanıcı tablosunda `ID` Primary Key'dir. Ancak kullanıcıların aynı E-posta adresiyle (`Email`) veya aynı TC Kimlik No ile iki kez kayıt olmasını istemezsin. `ID` benzersiz olsa bile, E-posta'nın da benzersiz olması gerekir. İşte bu iş kuralını (Business Rule) veritabanı seviyesinde `UNIQUE` ile sağlarsın.

---

### Deep Dive: PK ile Arasındaki Ezeli Rekabet

Mülakatlarda "Primary Key de benzersiz yapıyor, Unique de benzersiz yapıyor. Farkı ne?" sorusu %100 gelir. Bu tabloyu hafızana kazı:

1. **Adet Farkı:**
    
    - **Primary Key:** Bir tabloda **sadece 1 tane** olabilir. (Kral sadece bir tanedir).
        
    - **UNIQUE:** Bir tabloda **birden fazla** sütuna tanımlanabilir. (Hem E-posta benzersiz olsun, hem Telefon No benzersiz olsun diyebilirsin).
        
2. **NULL Durumu - _En Büyük Tuzak_:**
    
    - **Primary Key:** Asla `NULL` olamaz.
        
    - **UNIQUE:** `NULL` değer alabilir.
        
    - _SQL Server Detayı:_ SQL Server'da `UNIQUE` Constraint, varsayılan olarak **sadece 1 tane NULL** değere izin verir. İkinci NULL'u "tekrar eden veri" olarak görür ve hata verir. (Bazı diğer veritabanlarında çoklu NULL'a izin verilir).
        
3. **İndeksleme:**
    
    - **Primary Key:** Varsayılan olarak **Clustered Index** (Fiziksel Sıralama) oluşturur.
        
    - **UNIQUE:** Varsayılan olarak **Non-Clustered Index** (Mantıksal Sıralama/Kitap Arkası İndeksi) oluşturur. Bu da sorguları hızlandırır.
        

#### Composite Unique Constraint (Çoklu Sütun)

Bazen tek başına sütunlar tekrar edebilir ama ikililer tekrar edemez.

- **Senaryo:** Bir öğrenci `(OgrenciId)` bir dersi `(DersId)` birden fazla kez alabilir (kaldıysa). Ama aynı dönemde `(DonemId)` aynı dersi iki kere alamaz.
    
- **Kısıtlama:** `UNIQUE (OgrenciId, DersId, DonemId)`
    
- Bu üçlünün kombinasyonu benzersiz olmalıdır.
    

---

### Backend Developer İçin Neden Önemli?

1. **Hata Yönetimi (Exception Handling):** Kullanıcı kayıt formunda "Kaydet"e bastı. Backend kodun veritabanına `INSERT` atmaya çalıştı. Eğer o mail adresi zaten varsa, SQL Server **Error 2601** veya **2627** fırlatır.
    
    - Sen Backend Developer olarak `try-catch` bloğunda bu özel SQL hatasını yakalamalı ve kullanıcıya _"Bilinmeyen bir hata oluştu"_ yerine **"Bu e-posta adresi zaten kullanımda"** mesajını dönmelisin.
        
2. **Race Condition (Yarış Durumu) Çözümü:** Kod tarafında şöyle bir kontrol yapsan bile yetmez:
    
    C#
    
    ```csharp
    if (db.Users.Any(u => u.Email == email)) return "Hata";
    db.Users.Add(user);
    ```
    
    - Aynı anda (milisaniye farkla) iki kişi aynı maille kayıt olursa, `if` bloğu ikisi için de "Yok" der ve ikisini de eklemeye çalışır.
        
    - Veritabanındaki `UNIQUE Constraint`, bu durumda son kaledir. İkinci isteği reddeder ve veri bütünlüğünü korur.
        
3. **Entity Framework Core:** Veritabanını kodla tasarlarken (Code-First) bu kısıtlamayı şöyle eklersin:
    
    C#
    
    ```csharp
    modelBuilder.Entity<User>()
        .HasIndex(u => u.Email)
        .IsUnique();
    ```