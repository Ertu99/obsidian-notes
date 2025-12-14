

Çoğu yeni başlayan **Git** ile **GitHub**'ı aynı şey zanneder. Bu ayrımı yaparak başlayalım.

- **Git:** Arabanın motorudur. Yerel bilgisayarında çalışır.
    
- **GitHub/GitLab:** Otoparktır. Arabayı (projeni) park ettiğin, başkalarının gelip inceleyebildiği bulut alanıdır.
    


---

### 1. Neden "Hosting" Hizmetine İhtiyacımız Var?

Bilgisayarın bozulursa projen yok olur. Git, "Dağıtık" (Distributed) bir sistemdir demiştik. İşte bu hosting servisleri, projenin **"Remote" (Uzak)** bir kopyasını tutar.

Sadece yedekleme değil, 3 ana amaç vardır:

1. **Yedekleme:** Local Repo silinse bile Remote Repo durur.
    
2. **İşbirliği (Collaboration):** Takım arkadaşın senin bilgisayarına giremez ama GitHub'daki kodu indirip (`pull`) çalışabilir.
    
3. **CI/CD (Otomasyon):** Kod GitHub'a düştüğü anda otomatik testler çalışsın, sunucuya yüklensin istersin (Buna ileride geleceğiz).
    

---

### 2. Büyük Üçlü: GitHub vs GitLab vs Bitbucket

1. **GitHub:**
    
    - **Sahibi:** Microsoft.
        
    - **Özelliği:** Yazılım dünyasının "Facebook"udur. Open Source projelerin %99'u buradadır. Portfolyon burasıdır.
        
    - **Kullanım:** Bireysel geliştiriciler ve Open Source toplulukları için standarttır.
        
2. **GitLab:**
    
    - **Özelliği:** DevOps (Operasyon) süreçlerinde çok güçlüdür. Şirketler genellikle kendi sunucularına kurabildikleri için (Self-Hosted) güvenlik amacıyla tercih ederler.
        
3. **Bitbucket:**
    
    - **Sahibi:** Atlassian (Jira ve Trello'nun sahibi).
        
    - **Özelliği:** Eğer şirket proje takibi için Jira kullanıyorsa, entegrasyonu çok iyi olduğu için Bitbucket tercih edilir.
        

**Özet:** Sen GitHub kullanacaksın. Mülakatlarda GitHub profiline bakacaklar.

---

### 3. Kritik Kavramlar (Jargon)

GitHub kullanırken şu terimleri sürekli duyacaksın:

#### A. Repository (Repo)

Projenin saklandığı klasör. "GitHub'da repo açtım" demek, projeme bir sayfa oluşturdum demektir.

#### B. Fork (Çatallama)

Başkasına ait bir projeyi (örneğin Microsoft'un bir projesini), "Ben bunun üzerinde değişiklik yapacağım" diyerek kendi hesabına kopyalamaktır. Orijinal projeyi etkilemezsin.

#### C. Pull Request (PR) - **En Önemlisi**

Fork ettiğin projede bir geliştirme yaptın ve sahibine diyorsun ki: "Ben bir özellik ekledim, kodlarımı incele ve uygun görürsen kendi ana projene çek (Pull)."

Open Source dünyası ve kurumsal şirketler PR yöntemiyle çalışır. Kimse kafasına göre ana dala (main) kod atamaz. Kod incelemesi (Code Review) burada yapılır.

#### D. Issues

Projedeki hataların, isteklerin veya yapılacak işlerin listelendiği "bilet" (ticket) sistemidir.

---

### 4. Güvenlik: HTTPS vs SSH (Anahtarını Kaybetme)

GitHub'a kod gönderirken (`push`), GitHub senin gerçekten "Sen" olduğunu bilmek ister. İki yöntem vardır:

1. **HTTPS:** Her işlemde kullanıcı adı/şifre (veya token) sorar. Güvenlidir ama sürekli şifre girmek yorucudur.
    
2. **SSH (Secure Shell):** **Profesyonel yöntem.**
    
    - Bilgisayarında bir "Anahtar Çifti" (Key Pair) oluşturursun.
        
    - **Private Key (Özel Anahtar):** Bilgisayarında kalır, kimseye verilmez.
        
    - **Public Key (Genel Anahtar):** GitHub'a yüklenir.
        
    - Sen GitHub'a bağlandığında, GitHub elindeki kilit (Public) ile senin elindeki anahtarın (Private) uyup uymadığına bakar. Şifre sormaz.
        

**Öneri:** Hem Windows hem Mac kullanıyorsun. İkisinde de SSH anahtarı oluşturup GitHub ayarlarına eklemeni şiddetle tavsiye ederim.

---
Sen `commit` dediğinde her şey senin bilgisayarındaki `.git` klasörüne (Local Repo) yazılır.

Eğer `push` yapmadan bilgisayarın bozulursa, o kodlar **GitHub'a hiç gitmediği için** sonsuza dek kaybolur.

- **Commit:** "Bilgisayarıma kaydet."
    
- **Push:** "Buluta (GitHub'a) yükle."
    

Bu yüzden altın kural: **"İşin bittiğinde commit at, günün sonunda mutlaka push'la."**