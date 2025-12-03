## 1️⃣ Nedir?

> Normalizasyon, veritabanındaki gereksiz veri tekrarlarını (redundancy) önlemek
> 
> ve **veri tutarlılığını (consistency)** sağlamak için kullanılan tasarım kurallarıdır.

**Amaç:**

- Aynı bilgiyi birden fazla yerde tutmamak.
- Güncelleme, silme, ekleme hatalarını önlemek.
- Tabloları küçük, anlamlı parçalara bölmek.

---

## 💡 2️⃣ Gerçek hayat örneği

Diyelim ki şöyle bir tablo tasarladın 👇

|StudentId|StudentName|CourseName|TeacherName|
|---|---|---|---|
|1|Ali|Math|Ahmet|
|2|Ayşe|Physics|Mehmet|
|3|Ali|Physics|Mehmet|

### ❌ Problem:

- “Ali” iki kez geçiyor.
    
- “Mehmet” hem 2. hem 3. satırda var.
    
- Bir öğretmenin ismi değişirse her satırı güncellemen gerekir.
    
    (Yani **update anomaly** olur.)
    

### ✅ Çözüm:

Normalizasyon ile tabloyu **böl** 👇

- **Students(StudentId, StudentName)**
- **Courses(CourseId, CourseName, TeacherId)**
- **Teachers(TeacherId, TeacherName)**
- **StudentCourses(StudentId, CourseId)** (ilişki tablosu)

Artık “Mehmet” sadece **bir kez** kayıtlı.

Değişirse her yerde otomatik güncellenir.

---

## ⚙️ 3️⃣ Neden önemli?

|Problem|Normalizasyon çözümü|
|---|---|
|Veri tekrar eder|Her bilgi yalnızca bir yerde tutulur|
|Güncelleme hataları|Tüm tablo yerine tek kayıt değişir|
|Silme hataları|Bağlı kayıtlar korunur|
|Gereksiz alanlar|Tablo daha sade hale gelir|

---

## 🧱 4️⃣ Normalizasyon Aşamaları (Formlar)

Veritabanı normalizasyonu genelde **5 aşamada** tanımlanır ama mülakatlarda **3 tanesi** yeterlidir:

1️⃣ **1NF (Birinci Normal Form)**

2️⃣ **2NF (İkinci Normal Form)**

3️⃣ **3NF (Üçüncü Normal Form)**

(Opsiyonel: 4NF, 5NF, Boyce–Codd NF)

---

## 🔹 1NF – First Normal Form

> Her hücrede tek bir değer olmalı, tekrar eden gruplar olmamalı.

### ❌ Hatalı:

|Id|Name|Courses|
|---|---|---|
|1|Ali|Math, Physics|
|2|Ayşe|Biology|

### ✅ Doğru (1NF):

|Id|Name|Course|
|---|---|---|
|1|Ali|Math|
|1|Ali|Physics|
|2|Ayşe|Biology|

Yani hücre içinde “liste” tutulmaz, **her değer ayrı satır olur**.

## 2️⃣ 2NF Nedir?

> 2NF, bir tablonun 1NF’te olması ve
> 
> **her kolonun (özelliğin)** tablodaki **tüm birincil anahtara (primary key)** **tam bağlı** olmasını ister.

---

## 🧠 3️⃣ “Tam bağlı olmak” ne demek?

Bir tablonun **birden fazla kolon** içeren **birleşik (composite) primary key**’i varsa

örneğin `(StudentId, CourseId)` gibi,

her sütun bu ikisine **birlikte** bağlı olmalı.

Eğer bir sütun sadece birine bağlıysa, o tablo **2NF’i ihlal eder.**

---

## 💻 4️⃣ Hatalı (2NF ihlali yapan) tablo örneği

|StudentId|CourseId|StudentName|CourseName|TeacherName|
|---|---|---|---|---|
|1|10|Ali|Math|Ahmet|
|1|20|Ali|Physics|Mehmet|
|2|10|Ayşe|Math|Ahmet|

### 🔍 Buradaki Primary Key:

`(StudentId, CourseId)`

Çünkü bir öğrenci birden fazla derse girebilir,

ve her öğrenci–ders çifti **benzersizdir**.

---

### 🔍 Ama dikkat et:

|Kolon|Bağlı olduğu şey|Sorun|
|---|---|---|
|`StudentName`|sadece `StudentId`|🔴 sadece bir anahtar sütuna bağlı|
|`CourseName`|sadece `CourseId`|🔴 sadece bir anahtar sütuna bağlı|
|`TeacherName`|`CourseId` üzerinden bağlı|🔴 transit bağlılık var|

Yani bu tablo:

- 1NF’i sağlıyor (tek değerli)
- ama 2NF’i sağlamıyor çünkü sütunlar **anahtarın bir kısmına** bağlı.

---

## ✅ 5️⃣ 2NF’e uygun hale getirelim

Bu tabloyu parçalıyoruz 👇

**A) Students tablosu**

|StudentId|StudentName|
|---|---|
|1|Ali|
|2|Ayşe|

**B) Courses tablosu**

|CourseId|CourseName|TeacherName|
|---|---|---|
|10|Math|Ahmet|
|20|Physics|Mehmet|

**C) StudentCourses tablosu (ilişki tablosu)**

|StudentId|CourseId|
|---|---|
|1|10|
|1|20|
|2|10|

---

## 🧠 6️⃣ Artık tablo ne hale geldi?

|Tablo|Açıklama|
|---|---|
|**Students**|Öğrenciye özel bilgiler burada (sadece StudentId’ye bağlı)|
|**Courses**|Derse özel bilgiler burada (sadece CourseId’ye bağlı)|
|**StudentCourses**|Öğrenci–ders eşleşmeleri burada (ikisine bağlı)|

Artık:

- `StudentName` sadece `StudentId`’ye bağlı ✅
- `CourseName` sadece `CourseId`’ye bağlı ✅
- `StudentCourses` sadece “birlikte” anlamlı ✅

Dolayısıyla artık **2NF tamamdır.**

# THIRD NORMAL FORM (3NF) – Detaylı Anlatım ve Örnekler

---

## 🎯 1️⃣ Ön Bilgi – Önceki Aşamaların Mantığı

|Aşama|Amaç|
|---|---|
|**1NF**|Hücrelerde tek değer olmalı.|
|**2NF**|Her sütun, tüm birincil anahtara bağlı olmalı.|
|**3NF**|Her sütun **yalnızca anahtara** bağlı olmalı, **başka kolona değil.**|

Yani 3NF, 2NF’i geliştirip “**dolaylı (transitive)** bağımlılıkları” da ortadan kaldırır.

---

## ⚙️ 2️⃣ Tanım

> Bir tablo 2NF'te olmalı
> 
> ve
> 
> **anahtar olmayan bir kolon, başka bir anahtar olmayan kolona bağlı olmamalıdır.**

Bu cümle karışık gibi ama örnekle hemen oturacak 👇

---

## 💻 3️⃣ Hatalı (3NF ihlali yapan) tablo örneği

|StudentId|StudentName|DepartmentId|DepartmentName|
|---|---|---|---|
|1|Ali|1|Engineering|
|2|Ayşe|2|Physics|
|3|Mehmet|1|Engineering|

### 🔍 İnceleyelim:

- **Primary Key** → `StudentId`
    
- `DepartmentId` → `DepartmentName`’i belirliyor.
    
- Yani `DepartmentName`, `StudentId`'ye **doğrudan** değil,
    
    `DepartmentId` üzerinden **dolaylı (transitive)** olarak bağlı.
    

🧠 İşte bu, 3NF ihlali.

---

## ❌ Neden yanlış?

Eğer “Engineering” ismini değiştirmen gerekirse,

her “Engineering” satırını **tek tek** güncellemen gerekir.

Yani **veri tekrar ediyor** ve **update anomaly** oluşuyor.

---

## ✅ 4️⃣ 3NF’e uygun hale getirelim

Tabloyu ikiye ayırıyoruz 👇

### 🧱 Students tablosu

|StudentId|StudentName|DepartmentId|
|---|---|---|
|1|Ali|1|
|2|Ayşe|2|
|3|Mehmet|1|

### 🧱 Departments tablosu

|DepartmentId|DepartmentName|
|---|---|
|1|Engineering|
|2|Physics|

### 🔍 Artık:

- `DepartmentName`, doğrudan `DepartmentId`’ye bağlı ✅
- `DepartmentId`, `StudentId`’ye bağlı ✅
- Dolaylı bağımlılık kalmadı ✅