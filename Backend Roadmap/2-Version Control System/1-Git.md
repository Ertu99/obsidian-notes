**Git (Versiyon Kontrol Sistemi)**

Git, yazılım projelerindeki değişiklikleri zaman içinde takip eden, birden fazla geliştiricinin aynı proje üzerinde birbirini ezmeden çalışmasını sağlayan **dağıtık (distributed)** bir versiyon kontrol sistemidir.

Neden Kullanılır?

Kod yazarken hata yaptığında "Geri Al" (Undo) tuşu yetmez. Git, projenin her anının fotoğrafını çeker (Snapshot). İstediğin zaman 3 ay önceki koda dönebilir, kimin hangi satırı ne zaman ve neden değiştirdiğini görebilirsin. Ayrıca, takım çalışmasında kodların güvenli bir şekilde birleştirilmesini (Merge) sağlar.

---

### Deep Dive: Git Mimarisi ve Çalışma Mantığı (Bölüm 1)

Bir Junior Developer olarak Git komutlarını ezberlemek yetmez, arka planda verinin nasıl aktığını bilmelisin. Mülakatlarda "Commitlediğin ama Pushlamadığın kod nerede durur?" diye sorarlar.

#### 1. Dağıtık (Distributed) Ne Demek?

Eski sistemlerde (SVN gibi) kod tek bir ana sunucuda dururdu. Sunucu çökerse kimse çalışamazdı.

Git'te ise her geliştiricinin bilgisayarında projenin tam bir kopyası (tüm geçmişiyle birlikte) bulunur. İnternet olmasa bile kod geçmişine bakabilir, branch açabilir ve commit atabilirsin. İnternet gelince sunucuyla eşitlersin.

#### 2. Git'in Üç Temel Alanı (The Three States) - _Çok Önemli_

Git, dosyalarını 3 farklı alanda yönetir. Bu akışı kafanda oturtman şart:

1. **Working Directory (Çalışma Dizini):** Şu an üzerinde çalıştığın, kod yazdığın klasördür. Dosyalar buradayken henüz Git tarafından "paketlenmemiştir".
    
2. **Staging Area (Sahneleme Alanı / Index):** Commit'lemeden hemen önce dosyaları hazırladığın alandır. Markete gitmeden önce sepete attığın ürünler gibidir. (`git add` komutu buraya atar).
    
3. **Local Repository (Yerel Depo):** `.git` klasörüdür. Staging area'daki dosyaların paketlenip, mühürlenip tarihçeye kaydedildiği yerdir. (`git commit` komutu buraya atar).
    

#### 3. Temel Komutlar ve Arka Planı

- **git init:** Bir klasörü Git projesine çevirir. Gizli bir `.git` klasörü oluşturur. Tüm tarihçe bu klasördedir. Bunu silersen geçmiş silinir.
    
- **git clone [url]:** Uzaktaki (GitHub, GitLab) bir projeyi kendi bilgisayarına indirir.
    
- **git status:** "Hangi dosya değişti?", "Hangi dosya sahnede?" sorularının cevabını verir. En çok kullanacağın komuttur.
    
- **git add [dosya]:** Değişikliği **Working Directory**'den **Staging Area**'ya taşır. Git'e "Bu dosyayı bir sonraki pakete dahil et" dersin.
    
- **git commit -m "mesaj":** Staging Area'daki dosyaları alır, kalıcı bir "Snapshot" (fotoğraf) oluşturur ve **Local Repository**'ye kaydeder.
    
    - _Mülakat İpucu:_ Her commit'in benzersiz bir kimliği (HASH ID - SHA-1) vardır. Örn: `a1b2c3d...`
        

#### 4. Uzak Sunucuyla Konuşmak (Remote)

Buraya kadar her şey senin bilgisayarında (Local) oldu. Başkalarıyla çalışmak için:

- **git push:** Local Repository'deki commitleri Uzak Sunucuya (Remote Repository - Origin) gönderir.
    
- **git pull:** Uzak sunucudaki değişiklikleri alır ve senin kodunla birleştirir (Fetch + Merge).
    
- **git fetch:** Uzak sunucudaki değişiklikleri indirir ama senin koduna **karıştırmaz**. Sadece "neler değişmiş" diye bakmanı sağlar. (Güvenli olan budur).
    

#### 5. Branching (Dallanma) Mantığı

Git'in en büyük gücüdür. Ana proje (Main/Master) bozulmasın diye, her geliştirici kendine ait bir paralel evren (Branch) oluşturur.

- **Main (Master):** Canlıdaki çalışan, hatasız kodun olduğu ana daldır.
    
- **Feature Branch:** Sen "Login sayfası" yapacaksan `feature/login-page` diye bir dal açarsın. Ana daldan kopyalanır. Sen burada dünyayı yıksan bile ana proje etkilenmez. İşin bitince ana dalla birleştirirsin (**Merge**).
    

---

---

### Deep Dive: Git İleri Seviye (Part 2)

#### 1. Merge vs. Rebase (Mülakatın Yıldız Sorusu)

Her iki komut da "Bir daldaki kodları diğerine aktarmak" için kullanılır ama yöntemleri ve tarihçeye etkileri tamamen farklıdır.

- **Merge (Birleştirme):**
    
    - **Nedir:** İki dalın tarihçesini korur. İki dalı bir araya getiren yeni bir "Merge Commit" oluşturur.
        
    - **Avantajı:** Tarihçeyi (History) olduğu gibi saklar. Kimin ne zaman neyi birleştirdiği bellidir (Non-destructive).
        
    - **Dezavantajı:** Çok fazla dal varsa tarihçe "metro haritasına" döner, karmaşıklaşır.
        
- **Rebase (Taban Değiştirme):**
    
    - **Nedir:** Senin dalındaki commitleri alır, sanki hiç yan dal açmamışsın da ana dalın en sonundan devam etmişsin gibi tarihçeyi **yeniden yazar**.
        
    - **Avantajı:** Dümdüz, tertemiz, tek çizgi bir tarihçe oluşturur (Linear History).
        
    - **Tehlikesi (Bunu söylemen çok önemli):** **Asla** ortak çalışılan (public) bir dalda (Örn: Master/Main) Rebase yapılmaz! Çünkü tarihçeyi değiştirdiği için diğer geliştiricilerin kodunu bozar. Rebase sadece kendi yerel (local) dalında temizlik yapmak için kullanılır.
        

#### 2. Conflict (Çakışma) ve Çözümü

Mülakatta sorarlar: _"Conflict çıktı, ne yaparsın?"_

- **Senaryo:** Ahmet `HomeController.cs` dosyasının 10. satırını değiştirdi ve pushladı. Sen de aynı anda aynı dosyanın 10. satırını değiştirdin. Git, hangisinin doğru olduğunu bilemez ve işlemi durdurur.
    
- **Çözüm Adımları:**
    
    1. Dosyayı açarım. Git oraya `<<<<<<< HEAD` gibi işaretler koymuştur.
        
    2. Kodu incelerim. Ahmet'in kodu mu kalmalı, benimki mi, yoksa ikisinin birleşimi mi? Karar veririm.
        
    3. Gereksiz Git işaretlerini silerim ve kodu düzeltirim.
        
    4. `git add .` ve `git commit` yaparak işlemi tamamlarım.
        
- **Profesyonel Cevap:** "Conflict'ten korkmam, bu iletişimin bir parçasıdır. Gerekirse kodu yazan arkadaşımla konuşup doğru lojiği belirler ve merge işlemini tamamlarım."
    

#### 3. Değişiklikleri Geri Almak (Undo Things)

Hata yaptığında kullanacağın üç temel silah vardır:

- **git revert:** _En güvenli yoldur._ Hatalı commit'i silmez, o hatalı işlemi **tersine çeviren** yeni bir commit atar. Tarihçe bozulmaz. (Örn: "Hata" -> "Hatanın Tersi").
    
- **git reset:** _Tehlikelidir._ Zamanda yolculuk gibidir. Tarihçeyi siler.
    
    - `--soft`: Commit'i siler ama dosyalarındaki değişiklikleri korur (Staging area'da bekletir).
        
    - `--hard`: **Nükleer seçenektir.** Commit'i de siler, yazdığın kodları da siler. Her şey o tarihe geri döner. Geri dönüşü zordur.
        
- **git checkout:** Bir dosyayı en son commitlenen haline geri döndürür. (Örn: "Bu dosyayı çok kurcaladım bozdum, eskiye dönsün").
    

#### 4. Stashing (Zula/Saklama)

Çok sık yaşanan bir durum: Bir özellik üzerinde çalışıyorsun, kodların yarım yamalak, her yer hata veriyor. Patron geldi "Acil bir bug var, hemen düzeltmen lazım" dedi. Şu anki kodunu commit atamazsın çünkü bozuk.

- **git stash:** Çalıştığın tüm değişiklikleri geçici bir hafızaya (zula) alır ve çalışma masanı tertemiz yapar (son commit haline döndürür).
    
- Bug'ı düzeltir, gönderirsin.
    
- **git stash pop:** Zuladaki yarım kalan işlerini geri getirir ve kaldığın yerden devam edersin.
    

#### 5. .gitignore Dosyası (Bir .NET Developer İçin Hayati)

Git'in takip **etmemesi** gereken dosyaların listesidir. Mülakatta ".NET projesinde neleri ignore edersin?" diye sorabilirler.

- **Neleri Ignore Etmelisin?**
    
    - `/bin` ve `/obj` klasörleri: Bunlar derleme (build) sonucu oluşan dosyalardır. Kod değildir, gereksiz yer kaplar. Herkes kendi bilgisayarında oluşturabilir.
        
    - `.vs` klasörü: Visual Studio'nun senin kişisel ayarlarını tuttuğu klasördür.
        
    - `appsettings.Development.json` (Bazen): İçinde sana özel veritabanı şifreleri olabilir.
        
- **Neden?** Depoyu (Repository) çöplüğe çevirmemek ve güvenlik (şifreleri sızdırmamak) için.
    

---

### Backend Developer İçin Neden Önemli?

1. **CI/CD Pipeline:** Sen kodu `git push` yaptığın anda Azure DevOps veya GitHub Actions tetiklenir, projeyi derler ve sunucuya atar. Git, modern yazılım dağıtımının tetikleyicisidir.
    
2. **Code Review (Kod İnceleme):** Sen bir işi bitirince "Pull Request" (PR) açarsın. Takım liderin senin `diff`lerine (değişikliklerine) bakar. Temiz bir Git geçmişi, temiz bir kod incelemesi demektir.
    
3. **Versiyonlama:** Müşteri "Dün çalışan özellik bugün bozuldu" dediğinde, Git sayesinde tam olarak dünkü sürüme dönüp sorunu tespit edebilirsin.
    

