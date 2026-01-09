SQL öğrenme sürecinin "Ustalık" (Mastery) evresine hoş geldin. Şimdiye kadar kodun "Çalışması" ile ilgilendik, şimdiki hedefimiz "Hızlı Çalışması".

Bir Backend Developer olarak, uygulamanın yavaşlığı %90 ihtimalle veritabanı sorgularından kaynaklanır. API'nin 200ms yerine 5 saniyede cevap vermesinin suçlusu genellikle kötü yazılmış bir SQL sorgusudur. **Query Analysis**, bu suçluyu bulma dedektifliğidir.

Bu teknikleri 4 ana başlıkta inceleyelim:

---

### 1. EXPLAIN PLAN (Sorgunun Röntgeni)

Sen SQL'e bir sorgu yazdığında (`SELECT * FROM...`), veritabanı motoru (Engine) bu veriyi nasıl getireceğine dair bir "Plan" yapar. "İndeks mi kullanayım?", "Tabloyu baştan sona mı tarayayım?", "Önce hangi tabloyu birleştireyim?" gibi kararları verir.

- **Komut:** Çoğu veritabanında (PostgreSQL, MySQL, Oracle) sorgunun başına `EXPLAIN` yazarak bu planı görebilirsin. SQL Server'da ise `Ctrl + L` veya "Display Estimated Execution Plan" butonu kullanılır.
    
- **Ne Gösterir?** Sorgunun sonucunu göstermez; **maliyetini (Cost)** gösterir.
    
- **Analiz:** Ağaç yapısı şeklinde adımları görürsün.
    
    - **Table Scan / Index Scan:** Kötü haber. Tüm tabloyu okuyor demektir.
        
    - **Index Seek:** İyi haber. Nokta atışı veriyi buluyor demektir.
        
    - **Cost %:** Hangi adımın sisteme en çok yük bindirdiğini yüzdesel olarak gösterir. (Örn: %80 sıralama işlemine gidiyorsa, oraya odaklanmalısın).
        

### 2. İstatistikleri İzleme (`SET STATISTICS IO ON`)

Süre (Duration) yanıltıcıdır. Sunucu o an boşsa sorgu hızlı çalışır, doluyken yavaş çalışır. Gerçek performans ölçümü **Logical Reads** (Okunan Sayfa Sayısı) ile yapılır.

- **Komut:** SQL Server'da sorgudan önce `SET STATISTICS IO ON` ve `SET STATISTICS TIME ON` komutlarını çalıştır.
    
- **Çıktı:** Messages sekmesinde sana şunu der: _"Table 'Users'. Scan count 1, logical reads 5000."_
    
- **Yorum:** Eğer 10 satırlık veri için 5000 sayfa (Page) okuyorsa, indeksin yanlış veya eksiktir. Hedefimiz okuma sayısını (I/O) düşürmektir.
    

### 3. Profiling (Canlı İzleme)

Uygulaman çalışırken hangi sorguların veritabanını yorduğunu anlamak için kullanılır.

- **SQL Profiler / Extended Events:** Sunucuya gelen tüm sorguları anlık olarak yakalar.
    
- **Filtreleme:** "Süresi 3 saniyeden uzun süren (`Duration > 3000`) sorguları bana listele" diyerek sistemdeki darboğazları (Bottlenecks) bulursun.
    
- **N+1 Problemi Tespiti:** Profiler'ı açtığında aynı sorgunun ("Select * From Orders Where UserId = ...") alt alta 1000 kere geldiğini görürsen, Backend kodunda bir döngü hatası (N+1) olduğunu anlarsın.
    

### 4. Refactoring (Sorguyu Temizleme)

Analiz ettin ve sorunu buldun. Şimdi kodu düzeltme zamanı.

- **Subquery vs JOIN:** İç içe geçmiş alt sorgular (Nested Subqueries) genellikle yavaştır. Bunları `JOIN` yapısına veya `CTE` (Common Table Expression) yapısına çevirmek performansı artırabilir.
    
- **Veri Tipi Uyuşmazlığı (Implicit Conversion):** Tabloda `VARCHAR` olan bir alana `INT` parametre gönderirsen, SQL indeksi kullanamaz ve tüm tabloyu dönüştürmeye çalışır. Parametre tiplerini düzeltmen gerekir.
    

---

### Backend Developer İçin Neden Önemli?

1. **ORM Tuzağı (Entity Framework / Django):** Sen C# tarafında `context.Users.ToList()` yazarsın, çok masum görünür. Ama Profiler ile baktığında arkada milyonlarca satırlık veri çektiğini ve sunucuyu kilitlediğini görebilirsin. "Generated SQL"i (Oluşan SQL'i) analiz etmek zorundasın.
    
2. **Mülakat Sorusu:** _"Bir API çok yavaş çalışıyor, veritabanından şüpheleniyorsun. Nasıl analiz edersin?"_
    
    - **Cevap:** "Önce Execution Plan'a bakarım, Full Table Scan var mı kontrol ederim. Sonra IO istatistiklerine bakıp gereksiz okuma yapılıyor mu incelerim. Gerekirse eksik indeksleri eklerim."