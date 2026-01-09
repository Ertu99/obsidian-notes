Bu 4 komut, veritabanı oyununun "Save/Load" mekanizmasıdır. ACID kurallarını uygulamak için kullandığımız direksiyon ve fren pedalları bunlardır.

Verdiğin 4 temel komutu bir oyun senaryosu üzerinden ve teknik detaylarıyla inceleyelim:

---

### 1. BEGIN (veya BEGIN TRANSACTION)

**Motto:** "Otomatik Kaydetmeyi Kapat." Normalde SQL "Auto-Commit" modunda çalışır. Yani sen `UPDATE` yazdığın an veri değişir. `BEGIN` komutu veritabanına şunu söyler: _"Ben söyleyene kadar hiçbir şeyi kalıcı yapma, hafızada tut."_

- **Oyun Analojisi:** Zor bir Boss savaşına girmeden önceki o gergin an. Oyun burada "Geçici Kayıt" moduna geçer.
    

### 2. COMMIT (Onayla / Kaydet)

**Motto:** "Hepsini Diske Yaz." `BEGIN` ile başlattığın ve hafızada bekleyen tüm işlemleri onaylar ve fiziksel diske yazar. Bu noktadan sonra geri dönüş yoktur.

- **Oyun Analojisi:** Boss'u yendin ve "Save Game" yaptın. Artık elektrik kesilse de level atladın, geri alamazsın.
    
- **Dikkat:** Commit edilmemiş veriler (Dirty Read) diğer kullanıcılar tarafından görülemez (Isolation seviyesine göre değişir ama genelde böyledir). Commit ettiğin an herkes görür.
    

### 3. ROLLBACK (Geri Al)

**Motto:** "Her Şeyi İptal Et." İşlemlerin yarısında bir hata aldın veya vazgeçtin. `ROLLBACK` komutu, `BEGIN` anından itibaren yaptığın her değişikliği siler ve veritabanını işlem öncesindeki haline (eski haline) döndürür.

- **Oyun Analojisi:** Boss sana tek attı. "Game Over" ekranında "Try Again" dedin ve Boss'un kapısının önünde (BEGIN noktası) hiçbir şey olmamış gibi, canın full olarak yeniden doğdun.
    

### 4. SAVEPOINT (Ara Kayıt Noktası)

**Motto:** "Bölüm Sonu Canavarına Gelmeden Önceki Checkpoint." Büyük bir transaction içinde, hata olursa en başa dönmek yerine, sadece belli bir noktaya dönmek için kullanılır.

- **Senaryo:**
    
    1. `BEGIN` (Oyuna başla)
        
    2. `INSERT` (Zombileri öldürdün) -> **SAVEPOINT ZombilerOldu**
        
    3. `UPDATE` (Bölüm sonu canavarına daldın)
        
    4. **HATA!** (Canavar seni öldürdü)
        
    5. `ROLLBACK TO ZombilerOldu` (Oyuna en baştan başlamazsın, zombileri öldürmüş halinden devam edersin).
        

---

### Backend Developer İçin Neden Önemli?

Bu komutlar, Backend kodunun (C# / Python) hata yönetimi (Error Handling) iskeletidir.

**Örnek (Pseudo-Code):**

C#

```csharp
try
{
    db.BeginTransaction(); // BEGIN

    // Adım 1: Hesaptan para düş
    db.Execute("UPDATE Hesap SET Bakiye = Bakiye - 100 WHERE Id = 1");

    // Adım 2: Karşı tarafa ekle
    db.Execute("UPDATE Hesap SET Bakiye = Bakiye + 100 WHERE Id = 2");

    // Her şey yolunda, kaydet
    db.Commit(); // COMMIT
}
catch (Exception ex)
{
    // Bir hata oldu (Elektrik kesildi, bakiye yetersiz vs.)
    // Yapılan yarım yamalak işleri geri al
    db.Rollback(); // ROLLBACK
    Log(ex.Message);
}
```

Eğer bu yapıyı kurmazsan; para bir hesaptan düşer, diğerine gitmez ve veritabanın tutarsız (Inconsistent) hale gelir.

Transaction yönetimini (Begin, Commit, Rollback, Savepoint) tamamladık.