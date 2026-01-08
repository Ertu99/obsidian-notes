`HAVING`, SQL sorgularında `GROUP BY` ile gruplanmış ve özetlenmiş veriler üzerinde **filtreleme** yapmak için kullanılan komuttur.

**Neden Kullanılır?** `WHERE` komutu, veriler henüz gruplanmadan, ham halindeyken çalışır. Bu yüzden `WHERE` içinde `SUM`, `COUNT`, `AVG` gibi toplama fonksiyonlarını kullanamazsın. Eğer "Toplam sipariş tutarı 10.000 TL'den fazla olan müşterileri getir" demek istiyorsan, önce toplamı hesaplamalı (GROUP BY), sonra bu toplama göre filtrelemelisin. İşte bu "hesaplanmış veriyi filtreleme" işini `HAVING` yapar.

---

### Deep Dive: HAVING vs WHERE (Mülakatın Zirvesi)

Bu konu mülakatlarda "Bir SQL sorgusunda `WHERE` ve `HAVING` arasındaki fark nedir?" şeklinde %100 sorulur. Farkı sadece "Biri gruplar, biri satırlar" diye açıklamak Junior seviyesidir. Senior olmak için **Çalışma Sırası (Order of Execution)** ile açıklamalısın.

#### 1. Çalışma Zamanı Farkı

SQL motoru sorguyu şu sırayla işler:

1. **WHERE:** Satırları diskten okurken eler. (Daha gruplama yapılmamıştır).
    
    - _Örnek:_ `WHERE Year = 2024` (Sadece 2024 yılı satırlarını al).
        
2. **GROUP BY:** Kalan satırları gruplar ve özetler (Toplar, sayar).
    
    - _Örnek:_ `GROUP BY City` (Şehirlere göre topla).
        
3. **HAVING:** Oluşan özet tabloyu eler.
    
    - _Örnek:_ `HAVING SUM(Total) > 5000` (Toplamı 5000'den büyük olan grupları bırak, diğerlerini at).
        

#### 2. Kural: Aggregate Fonksiyonlar

- **WHERE:** İçinde `SUM()`, `COUNT()`, `MAX()` **kullanılamaz**.
    
    - _Hatalı:_ `SELECT City FROM Sales WHERE SUM(Total) > 1000` -> **PATLAR!**
        
- **HAVING:** Bu fonksiyonlar için yaratılmıştır.
    
    - _Doğru:_ `SELECT City FROM Sales GROUP BY City HAVING SUM(Total) > 1000`
        

#### 3. Birlikte Kullanım (Best Practice)

İkisini aynı sorguda kullanabilirsin ve bu performans için harikadır.

- **Senaryo:** "2024 yılında (WHERE), toplam satışı 100 adetten fazla olan (HAVING) ürünleri getir."
    
- **Mantık:**
    
    - Önce `WHERE` ile 2023, 2022 verilerini elersin. (Veri kümesi küçülür, hız artar).
        
    - Sonra kalan 2024 verisini `GROUP BY` ile gruplarsın.
        
    - En son `HAVING` ile toplamı 100'ün altındakileri atarsın.
        

SQL

```
SELECT ProductId, SUM(Quantity)
FROM Sales
WHERE Year = 2024          -- 1. Önce yılı filtrele (Satır bazlı)
GROUP BY ProductId         -- 2. Grupla
HAVING SUM(Quantity) > 100 -- 3. Sonuçları filtrele (Grup bazlı)
```

#### 4. Performans İpucu

Eğer filtreleyeceğin sütun bir Aggregate (Toplama) işlemi gerektirmiyorsa, onu `HAVING` yerine `WHERE` içinde yazmalısın.

- _Kötü:_ `GROUP BY City HAVING City = 'Istanbul'` (Önce hepsini gruplar, sonra İstanbul'u seçer. Gereksiz işlem).
    
- _İyi:_ `WHERE City = 'Istanbul' GROUP BY City` (Baştan sadece İstanbul'u alır ve gruplar).
    

---

### Backend Developer İçin Neden Önemli?

1. **LINQ Karşılığı:** C# tarafında LINQ yazarken bu ayrımı bilmek zorundasın.
    
    - `WHERE`: `.Where(x => x.Year == 2024)`
        
    - `HAVING`: `.GroupBy(x => x.Category).Where(g => g.Sum(s => s.Total) > 5000)`
        
    - Dikkat edersen LINQ'te ikisi de `.Where()` metodudur ama **yerleri** (`GroupBy`'dan önce mi sonra mı olduğu) SQL'deki `WHERE` mi `HAVING` mi olacağını belirler.
        
2. **Raporlama Servisleri:** Yazdığın API'lerde "Stok durumu kritik olan (Sayısı 10'dan az) kategorileri getir" gibi business kuralları hep `HAVING` ile çözülür.
    
3. **Hata Ayıklama:** "Invalid usage of aggregate function in WHERE clause" hatasını aldığında, hemen "Ah, `SUM` kullanmışım, bunu `HAVING`'e taşımalıyım" diyebilmelisin.