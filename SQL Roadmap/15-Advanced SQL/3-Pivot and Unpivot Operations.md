SQL yol haritamızın finalini, raporlama dünyasının en sevilen ama yazımı en çok kafa karıştıran konusuyla yapıyoruz.

Bu işlemleri Excel'deki **"Pivot Table"** (Özet Tablo) özelliğinin SQL komutu hali olarak düşünebilirsin. Verinin yönünü değiştirirler:

- **PIVOT:** Satırları sütuna çevirir (Dikeyden Yataya).
    
- **UNPIVOT:** Sütunları satıra çevirir (Yataydan Dikeye).
    

---

### 1. PIVOT (Satırdan Sütuna)

Raporlama ekranlarında kullanıcılar veriyi genellikle "Matrix" formatında görmek ister.

- **Ham Veri (Veritabanındaki Hali):** | Yıl | Ay | Satış | | :--- | :--- | :--- | | 2023 | Ocak | 100 | | 2023 | Şubat | 200 | | 2023 | Mart | 150 |
    
- **İstenen Rapor (PIVOT Hali):** | Yıl | Ocak | Şubat | Mart | | :--- | :--- | :--- | :--- | | 2023 | 100 | 200 | 150 |
    

**Mantık:** `PIVOT` işlemi yaparken mutlaka bir **Aggregation** (Toplama, Sayma vb.) fonksiyonu kullanmalısın. Çünkü aynı hücreye denk gelen birden fazla veri olabilir, SQL bunları nasıl birleştireceğini (Topla? Ortalamasını al?) bilmek ister.

**Sözdizimi (Syntax) İpucu:** PIVOT statik bir yapıdır. Yani sütun adlarını (`[Ocak], [Şubat]`) elle yazman gerekir. Eğer "Hangi ayların geleceğini bilmiyorum, dinamik olsun" dersen, az önce öğrendiğin **Dynamic SQL** konusunu kullanarak bu sorguyu oluşturman gerekir.

### 2. UNPIVOT (Sütundan Satıra)

Genellikle kötü tasarlanmış (Excel mantığıyla kurulmuş) tabloları düzeltmek (Normalize etmek) için kullanılır.

- **Kötü Tasarım Tablo:** `Satislar` tablosunda `OcakSatis`, `SubatSatis`, `MartSatis` diye yan yana sütunlar var. Bu tabloya sorgu atmak zordur.
    
- **UNPIVOT İle Dönüşüm:** Bu sütunları alıp tek bir `Ay` sütunu ve tek bir `Tutar` sütunu haline getirirsin. Böylece `GROUP BY Ay` diyerek analiz yapabilirsin.
    

---

### Backend Developer İçin Neden Önemli?

1. **Frontend Grafik Kütüphaneleri (Chart.js / Highcharts):** React veya Vue tarafında kullandığın grafik kütüphaneleri genellikle veriyi belirli bir formatta (Array of Objects) ister.
    
    - Bazen `[{ name: 'Ocak', value: 100 }, ...]` ister (Unpivoted).
        
    - Bazen `[{ yil: 2023, ocak: 100, subat: 200 }]` ister (Pivoted).
        
    - Veriyi SQL tarafında PIVOT/UNPIVOT ile hazırlayıp API'den hazır formatta dönmek, Frontend tarafında JavaScript ile "takla attırmaktan" (Data Manipulation) çok daha performanslıdır.
        
2. **Export to Excel:** Müşteri "Bana bu raporu Excel olarak indir" dediğinde, genellikle PIVOT edilmiş (Yıllar satırda, Aylar sütunda) halini ister. SQL'den bu formatta veri çekip doğrudan Excel kütüphanesine (EPPlus vb.) basmak işini çok kolaylaştırır.