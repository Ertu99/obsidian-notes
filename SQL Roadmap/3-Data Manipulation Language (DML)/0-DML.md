Harika, şimdi SQL'in en hareketli kısmına, **Data Manipulation Language (DML)** başlığına giriş yapıyoruz. Önceki DDL konusu "İnşaat" idi, DML ise o evin içindeki "Yaşam"dır.

**Data Manipulation Language (DML)**

DML (Veri İşleme Dili), veritabanı tablolarının **içindeki verilerle** oynamamızı sağlayan SQL alt kümesidir. Tablonun yapısını (duvarlarını) değiştirmeyiz; sadece içine veri ekler, var olanı okur, günceller veya sileriz.

**Neden Kullanılır?** Bir uygulama kullanırken yaptığın her işlem aslında bir DML komutudur. Instagram'a fotoğraf yüklemek (`INSERT`), profilini güncellemek (`UPDATE`), akışı yenileyip fotoğrafları görmek (`SELECT`) veya bir yorumu silmek (`DELETE`). Uygulamaları canlı kılan şey DML'dir.

---

### Deep Dive: DML'in Karakteristiği ve CRUD Mantığı

Mülakatlarda "DML nedir?" dendiğinde sadece komutları saymak yetmez. DML'in DDL'den (önceki konudan) en büyük farkı **Transaction (İşlem) Yönetimi**dir.

#### 1. CRUD Operasyonlarının Karşılığı

Yazılım dünyasında her şey **CRUD** (Create, Read, Update, Delete) üzerine kuruludur. DML komutları bu döngüyü sağlar:

- **Create (Oluştur):** `INSERT` komutu. (Veri ekler).
    
- **Read (Oku):** `SELECT` komutu. (Veriyi getirir). _Not: Bazı kaynaklar SELECT'i DQL (Data Query Language) olarak ayırsa da pratikte DML'in kralıdır._
    
- **Update (Güncelle):** `UPDATE` komutu. (Veriyi değiştirir).
    
- **Delete (Sil):** `DELETE` komutu. (Veriyi siler).
    

#### 2. Geri Alınabilirlik (Rollback) - _Kritik Fark_

DDL komutları (CREATE, DROP) genellikle "Auto-Commit"tir, yani çalıştırdığın an iş biter, geri dönüşü zordur. Ancak DML komutları **Transaction** içinde çalışır.

- **Senaryo:** Banka uygulamasında A kişisinden para düştün (`UPDATE`), ama B kişisine eklerken elektrik kesildi.
    
- **Çözüm:** DML olduğu için işlemi **ROLLBACK** (Geri Al) yapabilirsin. A'nın parası geri gelir. Veri güvenliği DML'in doğasında vardır.
    

#### 3. Trigger Tetikleme

`TRUNCATE` (DDL) komutunun aksine, `INSERT`, `UPDATE` ve `DELETE` komutları veritabanındaki **Trigger (Tetikleyici)** mekanizmalarını çalıştırır. Yani bir satırı sildiğinde "Silinenler" tablosuna log atmak istiyorsan DML kullanmak zorundasın.

---

### Backend Developer İçin Neden Önemli?

1. **İşimizin %90'ı:** Bir Backend Developer olarak günün çoğunda veritabanı şemasını değiştirmeyiz (DDL), ama sürekli veri okur ve yazarız (DML). Yazacağın API'lerin neredeyse tamamı DML komutlarını sarmalayan kodlardır.
    
2. **Entity Framework Core:** Sen C# tarafında `context.Users.Add(user)` deyip `SaveChanges()` çağırdığında, EF Core bunu arka planda `INSERT INTO...` DML komutuna çevirir. `SaveChanges()` metodu aslında bir DML Transaction'ı başlatır ve bitirir.
    
3. **Performans:** Kötü yazılmış bir DML sorgusu (örneğin Index kullanmayan bir `SELECT` veya `WHERE` koşulu olmayan bir `UPDATE`), uygulamanı kilitleyebilir.