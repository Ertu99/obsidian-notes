**GitHub**

GitHub, Git sürüm kontrol sistemini kullanan projeler için bulut tabanlı bir barındırma (hosting) hizmetidir. Microsoft tarafından satın alınmıştır ve yazılımcıların "Facebook"u gibidir.

Neden Kullanılır?

Git ile yerel bilgisayarında tuttuğun kodları internet ortamında yedeklemek, takımdaki diğer kişilerle paylaşmak ve ortak geliştirmek için kullanılır. Sadece kod deposu değil; proje yönetimi, hata takibi (issue tracking) ve kod inceleme (code review) süreçlerinin yönetildiği merkezdir. Mülakatta en önemli ayrım şudur: Git bir araçtır (araba), GitHub o arabanın park edildiği devasa bir garajdır.

---

### Deep Dive: GitHub'ın Teknik Özellikleri ve Kavramlar

Bir Junior Developer olarak mülakata girdiğinde "GitHub hesabın var mı?" sorusu %100 gelecektir. Buradaki terimlere hakim olman, takım çalışmasına hazır olduğunu gösterir.

#### 1. Repository (Repo)

Projenin dosyalarının, tüm geçmişinin ve Git verilerinin tutulduğu klasördür.

- **Public:** Herkes kodunu görebilir (Open Source).
    
- **Private:** Sadece sen ve yetki verdiğin kişiler görebilir (Şirket projeleri).
    
- **README.md:** Reponun kapak sayfasıdır. Projenin ne işe yaradığını ve nasıl çalıştırılacağını anlatan Markdown formatındaki dosyadır. Mülakatlarda iyi hazırlanmış bir README, kodun kendisi kadar değerlidir.
    

#### 2. Pull Request (PR) - _Mülakatın En Kritik Terimi_

Takım çalışmasının kalbidir.

- **Senaryo:** Sen `login-feature` dalında çalışmanı bitirdin. Bu kodları ana projeye (main) dahil etmek istiyorsun.
    
- **İşlem:** GitHub üzerinden bir **Pull Request (Çekme İsteği)** açarsın. Bu şu demektir: _"Ben kodumu bitirdim, lütfen değişikliklerimi inceleyin ve ana projeye çekin."_
    
- **Code Review:** Takım liderin veya kıdemli yazılımcı (Senior) senin PR'ını açar. Satır satır kodunu okur. "Burada hata var", "Değişken ismini düzelt" gibi yorumlar yapar. Sen düzeltince PR'ı onaylar (Approve) ve kodu birleştirir (Merge).
    

#### 3. Fork ve Clone

- **Clone:** Yetkin olan bir repoyu kendi bilgisayarına indirmektir.
    
- **Fork:** Yetkin **olmayan** (örneğin Google'ın yazdığı açık kaynak bir proje) bir projeyi, kendi GitHub hesabına kopyalamaktır. Artık o kopya senindir, üzerinde istediğini yapabilirsin. Değişiklik yapıp orijinal sahibine "Bunu projene ekle" demek istersen PR gönderirsin.
    

#### 4. GitHub Actions (CI/CD)

GitHub artık sadece depo değil, bir otomasyon merkezidir.

- **Nedir:** Repona her kod attığında (push) çalışan robotlardır.
    
- **Kullanım:** Sen kodu gönderdiğinde GitHub Actions otomatik olarak;
    
    1. Projeyi derler (Build).
        
    2. Testleri çalıştırır (Unit Tests).
        
    3. Her şey yeşilse (başarılıysa) kodu Azure'a veya AWS'ye yükler (Deploy).
        
- **Junior Etkisi:** "GitHub Actions ile basit bir CI pipeline'ı kurdum" dersen mülakatta +10 puan alırsın.
    

#### 5. Issues (Konular/Sorunlar)

Projedeki yapılacak işlerin, bulunan hataların (bug) veya yeni özellik isteklerinin listelendiği yerdir. Bir nevi "To-Do List"tir. Issue'lar PR'lar ile bağlanabilir (Örn: "Bu PR #45 numaralı hatayı çözer").

---

### Backend Developer İçin Neden Önemli?

1. **Portfolyo:** İş başvurusu yaparken CV'ne GitHub linkini koyarsın. İnsan Kaynakları değil ama Teknik Ekip oraya girer; kod yazış tarzına, commit mesajlarına ("bugfix" mi yazmışsın yoksa "Fixed null exception in UserServise" mi?) ve projelerinin düzenine bakar. Senin aynandır.
    
2. **Microsoft Entegrasyonu:** Sen bir .NET gelişticisisin. GitHub Microsoft'un olduğu için Visual Studio ve VS Code ile kusursuz çalışır. Tek tıkla repo oluşturabilir, Visual Studio içinden çıkmadan PR açabilirsin.
    
3. **Open Source Kültürü:** Başkalarının kodunu okumak, yazmaktan daha öğreticidir. GitHub üzerindeki popüler .NET projelerini (örneğin `nopCommerce` veya `eShopOnContainers`) inceleyerek mimari öğrenebilirsin.
    

