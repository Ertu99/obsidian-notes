Database Migration, en basit tabiriyle **Veritabanı Şemanızın (Tablolar, Sütunlar, İndeksler) Versiyon Kontrolüdür (Git for Database).**

Nasıl ki kaynak kodunu (Source Code) Git ile yönetiyor, her değişikliği `commit` ediyor ve geçmişe dönebiliyorsan; Migration araçları da veritabanındaki yapısal değişiklikleri kod dosyaları (scriptler) halinde saklar ve yönetir.

### Neden İhtiyacımız Var? (The Problem)

Migration kullanmayan bir ekipte şu senaryo yaşanır:

1. Sen lokalde `Users` tablosuna `PhoneNumber` sütunu eklersin. Kodunu günceller, Git'e atarsın.
    
2. Takım arkadaşın kodu çeker (`git pull`), projeyi çalıştırır ve **HATA!** "Column 'PhoneNumber' not found."
    
3. Sen ona "Abi SQL'e girip `ALTER TABLE...` yazsana" dersin.
    
4. Canlıya (Production) çıkarken bu SQL'i çalıştırmayı unutursunuz ve site çöker.
    

**Migration bu kaosu bitirir.** Sen değişikliği bir migration dosyası olarak oluşturursun, arkadaşın kodu çekip "Migrate et" dediğinde veritabanı otomatik güncellenir.

---

### Nasıl Çalışır? (The Mechanism)

Bir Migration sistemi temel olarak iki yöne hareket eder: **UP** (İleri) ve **DOWN** (Geri). Her migration dosyası bu iki talimatı içermek zorundadır.

1. **UP (Apply):** Veritabanını yeni versiyona taşır. (Örn: Tablo oluştur, sütun ekle).
    
2. **DOWN (Revert):** Yapılan işlemi geri alır. (Örn: Tabloyu sil, sütunu kaldır).
    

#### Teknik Altyapı: `_MigrationHistory` Tablosu

Migration aracı (Entity Framework, Django, Alembic, Flyway vb.), veritabanında hangi scriptlerin çalıştığını nereden bilir? Veritabanının içinde (genelde senin görmediğin) özel bir tablo tutar:

- **EF Core:** `__EFMigrationsHistory`
    
- **Django:** `django_migrations`
    
- **Flyway:** `flyway_schema_history`
    

Sen "Migrate" komutunu verdiğinde; araç bu tabloya bakar, projedeki migration dosyalarıyla karşılaştırır ve sadece **henüz çalışmamış olanları** sırasıyla çalıştırır.

---

### Kod Örnekleri (Real World Scenarios)

Backend dünyasında en yaygın iki yaklaşımı (C#/.NET ve Python) inceleyelim. Mantık %100 aynıdır, sadece syntax değişir.

#### Senaryo: `Products` Tablosuna `Stock` Sütunu Eklemek

**1. C# / Entity Framework Core (Code-First Yaklaşımı)**

C#'ta önce C# class'ını (Entity) değiştirirsin, sonra migration oluşturursun.

_Adım 1: Entity Güncelleme_

C#

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    // Yeni eklediğimiz alan
    public int Stock { get; set; } 
}
```

_Adım 2: Migration Oluşturma (Terminal)_ `dotnet ef migrations add AddStockToProducts`

_Adım 3: Oluşan Migration Dosyası (Otomatik Üretilir)_

C#

```csharp
public partial class AddStockToProducts : Migration
{
    // UP: İleriye gitmek için ne yapayım?
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<int>(
            name: "Stock",
            table: "Products",
            nullable: false,
            defaultValue: 0);
    }

    // DOWN: Geri almak (Undo) için ne yapayım?
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Stock",
            table: "Products");
    }
}
```

_Adım 4: Veritabanına Uygula_ `dotnet ef database update`


### Migration Türleri

1. **Schema Migrations:** Şimdiye kadar anlattıklarımızdır. Tablo yapısını (`CREATE`, `ALTER`, `DROP`) değiştirir.
    
2. **Data Migrations (Seed Data):** Yapıyı değil, **içindeki veriyi** değiştirmek için kullanılır.
    
    - _Örnek:_ "Ülke tablosuna varsayılan olarak Türkiye, ABD ve Almanya'yı ekle."
        
    - _Örnek:_ "Kullanıcıların `FullName` sütununu parçalayıp `Name` ve `Surname` sütunlarına dağıt."
        
    - Bu işlemler de migration dosyası olarak yazılır ki, canlıya çıkıldığında veriler otomatik düzelsin.
        

---

### Backend Developer İçin "Altın Kurallar"

Bu kurallar, seni gecenin 3'ünde "Production patladı" diye aranmaktan kurtarır.

1. **Asla Geçmiş Migration Dosyasını Düzenleme:** Eğer bir migration dosyasını (`001_create_users`) commit ettiysen ve takım arkadaşların bunu aldıysa, o dosyayı bir daha **ASLA** elleme.
    
    - _Hata:_ "Ya `Users` tablosunda `Age` sütununu unutmuşum, dur şu eski dosyaya ekleyivereyim."
        
    - _Sonuç:_ Arkadaşının veritabanında o migration zaten çalışmış göründüğü için (History tablosu), senin eklemeni görmez. Veritabanlarınız uyumsuz hale gelir (Conflict).
        
    - _Çözüm:_ Her zaman yeni bir migration oluştur (`002_add_age_to_users`).
        
2. **İsimlendirme Hayat Kurtarır:** Migration dosyalarına `migration_1`, `migration_2` gibi isimler verme.
    
    - _Kötü:_ `20231025_update.cs`
        
    - _İyi:_ `20231025_AddStockColumnToProducts.cs`
        
    - Bir yıl sonra geçmişe baktığında neyin ne zaman değiştiğini anlaman gerekir.
        
3. **Nullable (Boş Geçilebilir) Kuralı:** İçi dolu devasa bir tabloya (`Users` - 1 Milyon Kayıt) yeni bir sütun eklerken, o sütunu **Zorunlu (Not Null)** yapamazsın!
    
    - Çünkü veritabanı, eski 1 milyon kayıt için o sütuna ne yazacağını bilemez ve patlar.
        
    - _Strateji:_ Önce sütunu `Nullable` olarak ekle veya `Default Value` (Varsayılan Değer) ver. Sonra içini doldur (`Data Migration`). En son `Not Null` yap.
        
4. **Production Öncesi Backup:** Migration komutunu canlı sunucuda (Production) çalıştırmadan önce **mutlaka** veritabanı yedeği alınmalıdır. `DOWN` metodu olsa bile, bazen veri kayıpları geri döndürülemez (Örneğin bir sütunu düşürmek - `DROP COLUMN`).