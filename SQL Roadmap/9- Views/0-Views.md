View (Görünüm), veritabanında fiziksel olarak yer kaplamayan (Materialized hariç), sadece bir SQL sorgusunun **kaydedilmiş halinden** ibaret olan **sanal tablodur**.

**Neden Kullanılır?** Bilgisayarının masaüstündeki "Kısayol" ikonları gibidir. Kısayola tıkladığında asıl dosya açılır. View'a sorgu attığında da arka planda asıl tablolar sorgulanır. Karmaşık ve uzun SQL sorgularını her seferinde tekrar yazmamak için onları bir isim altında (View) paketleriz.

---

### Deep Dive: Fiziksel vs. Sanal Ayrımı

Bir Junior Developer olarak tablolarla çalışmaya alışkınsın. View'lar ise senin hayatını kolaylaştıran "Maskelerdir".

#### 1. Karmaşıklığı Gizleme (Abstraction)

Diyelim ki her raporlamada 5 farklı tabloyu (Users, Orders, OrderDetails, Products, Categories) birleştirmen (`JOIN`) gerekiyor.

- **View Olmadan:** Her seferinde 20 satırlık SQL sorgusu yazarsın.
    
- **View İle:** Bu sorguyu `Vw_SatisRaporu` adıyla kaydedersin. Artık sadece `SELECT * FROM Vw_SatisRaporu` yazman yeterlidir. Arka plandaki 5 tabloluk karmaşayı görmezsin.
    

#### 2. Güvenlik Katmanı (Security)

Veritabanında `Personel` tablosunda herkesin maaşı (`Salary`) ve TC No'su yazıyor. Stajyerlerin bu tabloyu görmesi lazım ama maaşları görmemeli.

- **Çözüm:** `CREATE VIEW Vw_StajyerPersonel AS SELECT Ad, Soyad, Departman FROM Personel`
    
- Stajyerlere sadece bu View üzerinde yetki verirsin. Ana tabloya erişemezler, böylece maaş sütununu asla göremezler.
    

#### 3. Depolama Maliyeti Yoktur

Standart bir View, veriyi kendi içinde tutmaz. Sen View'ı çağırdığında, o an gidip ana tablolardan **canlı (real-time)** veriyi çeker. Yani ana tabloda bir değişiklik olduğunda View anında güncellenir.

#### 4. Materialized Views (Mülakat Sorusu)

Prompt'unda geçen "Materialized View" kavramı çok kritiktir.

- **Standart View:** Veriyi tutmaz, her çalıştırıldığında sorguyu baştan hesaplar. (Yavaştır ama günceldir).
    
- **Materialized View (Indexed View):** Sorgu sonucunu **fiziksel olarak diske yazar**. Tıpkı gerçek bir tablo gibi yer kaplar.
    
    - _Avantajı:_ Çok karmaşık raporları milisaniyede getirir (Çünkü önceden hesaplayıp kaydetmiştir).
        
    - _Dezavantajı:_ Ana tablo değiştiğinde bunun da güncellenmesi gerekir (Bakım maliyeti).
        

---

### Backend Developer İçin Neden Önemli?

1. **Refactoring (Kod Değişimi):** Veritabanı şemasını değiştirdin diyelim (Tablo adı değişti). Eğer kodunda (C#) direkt tabloya gidiyorsan, kod patlar. Ama arada bir View varsa, sadece View'ın içindeki sorguyu güncellersin, C# koduna dokunmana gerek kalmaz. View bir tampon bölge (Buffer) görevi görür.
    
2. **Entity Framework Core:** EF Core'da View'ları da tıpkı tablolar gibi bir Class (Entity) olarak tanımlayabilirsin (`.ToView("ViewName")`). Bu sayede karmaşık SQL sorgularını veritabanına yıkıp, C# tarafında temiz bir liste olarak çekersin.
    
3. **Legacy (Eski) Sistemler:** Eski ve kötü tasarlanmış bir veritabanıyla çalışmak zorundaysan, o tabloları düzeltmek yerine, onların üzerine temiz isimlendirilmiş View'lar kurarak kendi "Sanal Modern Veritabanını" oluşturabilirsin.