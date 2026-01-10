SQL yol haritasının son düzlüğüne, "Advanced" (İleri Seviye) bölüme hoş geldin. Buradan sonrası artık sadece sorgu yazmak değil, veritabanı motoruna **kod yazdırmaktır**.

**Dynamic SQL Nedir?** En basit tabiriyle: **"Kod yazan kod"** demektir. Normalde (Static SQL) sorgular sabittir; sütunlar, tablo adları bellidir. Dinamik SQL'de ise sorgu metnini (`String`) programın çalışma anında (Runtime), kullanıcının seçimlerine göre "LEGO parçaları gibi" birleştirerek oluşturursun ve çalıştırırsın.

---

### Neden İhtiyacımız Var? (E-Ticaret Senaryosu)

Bir alışveriş sitesinin filtreleme ekranını düşün.

- Kullanıcı bazen sadece "Fiyat Aralığı" seçer.
    
- Bazen "Marka" + "Renk" seçer.
    
- Bazen "Puan" + "Fiyat" + "Kargo Bedava" seçer.
    

**Static SQL ile Çözüm (Kâbus):** Tüm ihtimalleri `IF-ELSE` veya karmaşık `OR` mantığıyla yazmaya çalışırsan, yüzlerce satır kod çıkar ve yönetilemez hale gelir.

**Dynamic SQL ile Çözüm:** Sorguyu parça parça inşa edersin.

1. Temel: `SELECT * FROM Urunler WHERE 1=1`
    
2. Fiyat seçtiyse ekle: `+ " AND Fiyat < 100"`
    
3. Renk seçtiyse ekle: `+ " AND Renk = 'Kırmızı'"`
    
4. Sonuç: Tek ve temiz bir sorgu.
    

---

### Nasıl Yazılır? (T-SQL Örneği)

SQL Server'da iki yöntem vardır. Biri çok tehlikeli, diğeri profesyoneldir.

#### 1. Yöntem: `EXEC` (Kötü ve Tehlikeli)

String birleştirme yaparak çalıştırır. SQL Injection'a kapı açar.

SQL

```sql
DECLARE @Marka NVARCHAR(50) = 'Apple';
DECLARE @SQL NVARCHAR(MAX);

-- String birleştirme (String Concatenation)
SET @SQL = 'SELECT * FROM Urunler WHERE Marka = ''' + @Marka + '''';

EXEC(@SQL); -- Metni çalıştır
```

#### 2. Yöntem: `sp_executesql` (Güvenli ve Performanslı)

Backend Developer olarak **her zaman** bunu kullanmalısın. Parametre desteği vardır.

SQL

```sql
DECLARE @SQL NVARCHAR(MAX);
DECLARE @MarkaParam NVARCHAR(50) = 'Apple';

SET @SQL = N'SELECT * FROM Urunler WHERE Marka = @MarkaIceri';

-- Sorguyu parametreyle çalıştır (Güvenli)
EXEC sp_executesql @SQL,
     N'@MarkaIceri NVARCHAR(50)', -- Parametre tanımı
     @MarkaIceri = @MarkaParam;   -- Değer ataması
```

---

### Backend Developer İçin Kritik Noktalar

#### 1. SQL Injection Riski

Dynamic SQL'in en büyük dezavantajı güvenliktir.

- Eğer kullanıcı `@Marka` yerine `' OR '1'='1` gönderirse ve sen 1. yöntemi (String birleştirme) kullanıyorsan, hacker tüm ürünleri görür veya veritabanını silebilir.
    
- **Kural:** Dinamik SQL yazarken asla değişkenleri string içine `+` ile gömme. Mutlaka `sp_executesql` ile parametre olarak gönder.
    

#### 2. ORM Araçları (Entity Framework / Dapper)

Aslında sen C#'ta `var users = context.Users.Where(u => u.Age > 18);` yazdığında, EF Core arka planda senin için **Dynamic SQL** üretir.

- Senin yazdığın C# kodunu (Expression Tree) alır, çalışma anında (Runtime) uygun SQL metnine çevirir ve parametrelerle sunucuya gönderir. Yani ORM'ler, Dynamic SQL'in en büyük müşterisidir.
    

#### 3. Execution Plan Önbelleği (Plan Cache)

Static SQL'de veritabanı planı bir kere yapar ve saklar (Hızlıdır). Dynamic SQL'de sorgu metni sürekli değiştiği için (bir boşluk bile değişse), veritabanı her seferinde yeniden plan yapmak zorunda kalabilir. Bu da CPU kullanımını artırabilir. `sp_executesql` parametre kullandığı için bu sorunu minimize eder.

**Özetle:** Dynamic SQL, esneklik (Flexibility) sağlar ama güvenlik (Security) ve bakım (Maintenance) maliyeti getirir. Sadece gerçekten gerektiğinde (dinamik filtreleme, dinamik tablo adı vb.) kullanılmalıdır.