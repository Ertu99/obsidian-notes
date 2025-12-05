
### 1. Git Nedir? (Felsefesi ve Mimarisi)

Git, "Distributed Version Control System" (Dağıtık Versiyon Kontrol Sistemi) olarak geçer.

- **Neden "Dağıtık"?** Eski sistemlerde (SVN gibi) bir ana sunucu vardı, herkes oraya bağlanırdı. Sunucu çökerse iş biterdi. Git'te ise projeyi bilgisayarına indirdiğinde (`clone`), projenin **tüm tarihçesini**, ilk satırından son satırına kadar kendi diskine kopyalarsın. Şu an GitHub batsa, senin bilgisayarındaki kopyadan her şey kurtarılabilir.
    
- **Snapshot (Şipşak) Mantığı:** Çoğu versiyon sistemi "değişiklikleri" (delta) kaydeder. Git ise her "Save" (commit) anında projenin o anki halinin **tam bir fotoğrafını (snapshot)** çeker ve saklar. Değişmeyen dosyaları tekrar kopyalamaz, sadece eski fotoğrafa bir link (pointer) atar. Bu yüzden çok hızlıdır.
    

---

### 2. Git'in 3 Temel Bölgesi (The Three States)

Git'i anlamayanların %90'ı burayı bilmediği için hata yapar. Bir dosya Git evreninde 3 farklı durumda olabilir:

1. **Working Directory (Çalışma Alanı):** Şu an editörde (VS Code/Rider) görüp düzenlediğin yer. Dosyalar burada "Modified" (Değiştirilmiş) durumdadır ama Git henüz bunları kaydetmemiştir.
    
2. **Staging Area / Index (Sahne):** Burası bir bekleme odasıdır. Fotoğraf çekilmeden önce "Hangi dosyalar fotoğrafa girecek?" diye seçtiğin alandır.
    
3. **Local Repository (Yerel Depo):** `.git` klasörünün içidir. Fotoğrafın çekildiği ve veritabanına sonsuza kadar işlendiği yerdir.
    

Akış:

Kod Yaz (Working Dir) -> git add -> Hazırla (Staging) -> git commit -> Paketle (Repo).

---

### 3. Kurulum ve İlk Ayar (Config)

Git'i kurduktan sonra (Windows veya Mac), ona kim olduğunu söylemelisin. Çünkü her yapılan işlem (commit), bir yazar imzası taşır.

Terminali aç ve şu komutları gir (Bunları bir kez yapman yeterli):

Bash

```
git config --global user.name "Senin Adın"
git config --global user.email "email@adresin.com"
```

- `--global`: Bu ayarın bilgisayardaki tüm projeler için geçerli olacağını söyler.
    
- `--local`: Sadece o anki proje için geçerli kılar (Örn: Şirket projesinde iş mailini, kişisel projede şahsi mailini kullanmak istersen).
    

---

### 4. Temel Komutlar ve Yaşam Döngüsü

Bir .NET projesi üzerinden gidelim.

#### A. Başlatma (`init` veya `clone`)

Ya sıfırdan başlarsın ya da var olanı kopyalarsın.

- `git init`: Bulunduğun klasörü bir Git projesine dönüştürür. Gizli bir `.git` klasörü oluşturur.
    
- `git clone <url>`: Uzaktaki (GitHub/GitLab) bir projeyi indirir. `git init` yapmana gerek kalmaz, içinde gelir.
    

#### B. Durumu Kontrol Etme (`status`)

**En çok kullanacağın komut.** "Şu an hangi dosya nerede? Sahneye alındı mı? Değişti mi?" sorularının cevabıdır.

Bash

```
git status
```

#### C. Sahneye Alma (`add`)

Dosyaları Working Directory'den Staging Area'ya taşır.

- `git add DosyaAdi.cs`: Tek bir dosyayı ekler.
    
- `git add .`: (Nokta) O anki klasördeki **her şeyi** sahneye atar.
    

#### D. Kaydetme (`commit`)

Sahnedeki dosyaların fotoğrafını çeker ve veritabanına yazar.

Bash

```
git commit -m "Login ekranı eklendi"
```

- `-m`: Mesaj parametresidir. Mesajsız commit atamazsın.
    
- **Önemli:** Commit işlemi internet gerektirmez. Sadece kendi bilgisayarına (Local Repo) kayıt yaparsın.
    

#### E. Geçmişe Bakma (`log`)

Projenin tarihçesini gösterir.

Bash

```
git log
# Veya daha okunaklı hali (tek satırda):
git log --oneline
```

Burada gördüğün o karmaşık sayılar (örn: `a1b2c3d...`) **SHA-1 Hash**'tir. Her commit'in benzersiz kimlik numarasıdır.

---

### 5. Branching (Dallanma) - Git'in Süper Gücü

Burası detaylı bilinmeli. Diyelim ki proje "Canlı"da çalışıyor (Main/Master branch). Sen yeni bir "Ödeme Sistemi" ekleyeceksin. Ana kodu bozmamak için paralel bir evren yaratırsın.

- `git branch`: Mevcut dalları listeler.
    
- `git branch feature-odeme`: "feature-odeme" adında yeni bir dal yaratır (ama oraya geçmez).
    
- `git checkout feature-odeme` veya (yeni komut) `git switch feature-odeme`: O dala geçiş yapar. Artık yaptğin değişiklikler ana projeyi etkilemez.
    

Birleştirme (Merging):

İşin bitti, kodlar test edildi. Şimdi ana projeye (main) dahil etmelisin.

1. Önce ana dala geç: `git switch main`
    
2. Diğer dalı buraya çek: `git merge feature-odeme`
    

---

### 6. Uzak Sunucu (Remote) ile Çalışma

GitHub, GitLab gibi yerler "Remote" (Uzak) depodur.

- `git remote add origin <url>`: Yerel projenle GitHub'daki boş projeyi birbirine bağlar. "Origin", uzak sunucunun takma adıdır.
    
- `git push -u origin main`: Local'deki commit'lerini uzaktaki sunucuya (Upload) gönderir.
    
- `git pull`: Uzaktaki değişiklikleri kendi bilgisayarına çeker (Download + Merge).
    

---

### 7. Kritik Dosya: `.gitignore`

Bir .NET developer için hayati önem taşır.

Projeyi derlediğinde (dotnet build), bin ve obj klasörleri oluşur. Bunlar gereksiz dosyalar (çöp) ve makineye özeldir. Bunları Git'e asla atmamalısın.

Projenin ana dizinine `.gitignore` adında bir dosya açıp içine şunları yazmalısın:

Plaintext

```
bin/
obj/
.vscode/
*.user
```

Git artık bu dosyaları görmezden gelir. "git add ." yapsan bile bunları almaz.

---

### 8. HEAD Kavramı (Detay Bilgi)

Git loglarına baktığında HEAD -> main gibi bir yazı görürsün.

HEAD, "Şu an nereye bakıyorsun?" sorusunun cevabıdır. Bir kasetçalar iğnesi gibidir. Normalde en son commit'i gösterir. Ama istersen HEAD'i geçmişteki bir commit'e taşıyıp (Checkout), projenin 2 yıl önceki haline ışınlanabilirsin. Buna "Detached HEAD" durumu denir.

---

### Uygulamalı Pratik (Windows/Mac Terminalinde Yapabilirsin)

Şu senaryoyu zihninde veya terminalde canlandır:

1. `git init`: Projeyi başlattın.
    
2. `Program.cs` dosyasını yarattın ve kod yazdın.
    
3. `git status`: Dosya kırmızı renkte (Untracked) görünür.
    
4. `git add .`: Dosya yeşil renkte (Staged) görünür.
    
5. `git commit -m "Initial commit"`: Dosya listeden kaybolur (çünkü artık depolandı, yapılacak iş kalmadı).
    

---

