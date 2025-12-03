# DATABASE RELATIONSHIPS (İLİŞKİ TÜRLERİ)

---

## 🎯 1️⃣ Nedir?

> Veritabanında iki tablo arasındaki bağı ifade eder.
> 
> Amaç: tabloları **mantıksal olarak bağlamak**, verinin **bütünlüğünü** korumak.

İlişkiler genelde **primary key** ve **foreign key** alanlarıyla kurulur.

---

## 🧱 2️⃣ Temel Kavramlar

|Kavram|Açıklama|
|---|---|
|**Primary Key (PK)**|Tablo içindeki her satırı benzersiz tanımlar.|
|**Foreign Key (FK)**|Başka bir tablodaki **Primary Key’e referans** verir.|
|**Relation (İlişki)**|PK ↔ FK bağlantısıdır.|

---

## 🔗 3️⃣ İlişki Türleri

### 1️⃣ One-to-One (1–1)

### 2️⃣ One-to-Many (1–N)

### 3️⃣ Many-to-Many (N–N)

---

## 🔹 1️⃣ ONE-TO-ONE (Birebir İlişki)

> Bir tablodaki bir kayıt, diğer tablodaki tam bir kayıtla eşleşir.
> 
> Yani her iki tarafta da **tek bir eş** vardır.

### 🎓 Örnek:

Bir öğrencinin **tek bir kimlik kartı** olabilir.

|Students|IdentityCards|
|---|---|
|Id (PK)|StudentId (FK, Unique)|
|Name|CardNumber|

```sql
CREATE TABLE Students (
    Id INT PRIMARY KEY,
    Name NVARCHAR(50)
);

CREATE TABLE IdentityCards (
    Id INT PRIMARY KEY,
    CardNumber NVARCHAR(50),
    StudentId INT UNIQUE FOREIGN KEY REFERENCES Students(Id)
);

```

🧠 `UNIQUE` kısıtı sayesinde her öğrenciye **tek kart** düşer.

---

## 🔹 2️⃣ ONE-TO-MANY (Bire-Çok İlişki)

> Bir tablodaki bir kayıt, diğer tablodaki birden fazla kayıtla ilişkili olabilir.
> 
> En yaygın ilişki türüdür.

### 🏢 Örnek:

Bir departmanda **birden fazla çalışan** olabilir.

|Departments|Employees|
|---|---|
|Id (PK)|Id (PK)|
|DepartmentName|FirsName|
||DepartmentId (FK)|

```sql
CREATE TABLE Departments (
    Id INT PRIMARY KEY,
    DepartmentName NVARCHAR(50)
);

CREATE TABLE Employees (
    Id INT PRIMARY KEY,
    FirsName NVARCHAR(50),
    DepartmentId INT FOREIGN KEY REFERENCES Departments(Id)
);

```

### 🔍 Ne olur?

- “IT” departmanının birçok çalışanı olabilir.
- Ama bir çalışan **sadece bir departmanda** bulunur.

---

## 🔹 3️⃣ MANY-TO-MANY (Çoktan Çoğa İlişki)

> Bir tablodaki birden fazla kayıt, diğer tablodaki birden fazla kayıtla ilişkili olabilir.

### 🎓 Örnek:

Bir öğrenci **birden fazla derse** girebilir,

ve bir dersin **birden fazla öğrencisi** olabilir.

|Students|Courses|StudentCourses|
|---|---|---|
|Id (PK)|Id (PK)|StudentId (FK)|
|Name|CourseName|CourseId (FK)|

```sql
CREATE TABLE Students (
    Id INT PRIMARY KEY,
    Name NVARCHAR(50)
);

CREATE TABLE Courses (
    Id INT PRIMARY KEY,
    CourseName NVARCHAR(50)
);

CREATE TABLE StudentCourses (
    StudentId INT FOREIGN KEY REFERENCES Students(Id),
    CourseId INT FOREIGN KEY REFERENCES Courses(Id),
    PRIMARY KEY (StudentId, CourseId)
);

```

### 🔍 Ne olur?

- `StudentCourses` aradaki **bağlantı (junction)** tablosudur.
    
- Her öğrenci birden fazla derse,
    
    her ders de birden fazla öğrenciye bağlanabilir.
    

---

## 💬 4️⃣ İlişki Türlerini Özetleyelim

|İlişki Türü|Açıklama|Örnek|
|---|---|---|
|One-to-One|Her iki tabloda tek karşılık|Öğrenci – Kimlik Kartı|
|One-to-Many|Bir kayıt, diğer tabloda birçok kayıtla ilişkili|Departman – Çalışan|
|Many-to-Many|Her iki taraf da çoklu ilişki kurabilir|Öğrenci – Ders|

---

## 🧠 5️⃣ İlişki Türü Seçme Rehberi

|Soru|Cevap|Tür|
|---|---|---|
|“Bir öğe sadece bir eşe mi sahip olacak?”|Evet|One-to-One|
|“Bir öğe birçok elemana sahip olabilir mi?”|Evet|One-to-Many|
|“Her iki taraf da çoklu mu?”|Evet|Many-to-Many|

---

## ⚙️ 6️⃣ EF Core’daki Karşılıkları (kısa bilgi)

|SQL İlişkisi|EF Core Kod Karşılığı|
|---|---|
|One-to-One|`HasOne().WithOne()`|
|One-to-Many|`HasOne().WithMany()`|
|Many-to-Many|`HasMany().WithMany()`|

Örnek:

```csharp
modelBuilder.Entity<Employee>()
    .HasOne(e => e.Department)
    .WithMany(d => d.Employees)
    .HasForeignKey(e => e.DepartmentId);

```

---

## ✅ 7️⃣ Not Defterine Yazmalık Özet

> 🔹 Veritabanı ilişkileri, tablolar arasında veri bütünlüğü sağlar.
> 
> 🔹 **Primary Key** benzersiz kimliktir, **Foreign Key** başka tablodaki anahtara referans verir.
> 
> 🔹 3 ana tür vardır:
> 
> 1️⃣ One-to-One → Tek – Tek
> 
> 2️⃣ One-to-Many → Tek – Çok (en yaygın)
> 
> 3️⃣ Many-to-Many → Çok – Çok (bağlantı tablosu gerekir)
> 
> 🔹 EF Core’da bu ilişkiler `HasOne`, `WithMany`, `WithOne`, `HasMany` gibi metotlarla kurulabilir.