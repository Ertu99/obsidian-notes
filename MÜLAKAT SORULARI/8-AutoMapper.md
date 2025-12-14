## 🎯 Tanım:

**AutoMapper**, .NET’te bir **nesneyi başka bir nesneye otomatik olarak dönüştüren kütüphanedir.**

Yani:

> “Property isimleri aynı olan iki sınıfı,
> 
> tek satırda birbirine kopyalamanı sağlar.”

---

## 🔹 Neden Gerekli?

Normalde `DTO ↔ Entity` dönüşümünü elle yapmak zahmetlidir:

```csharp
Hotel hotel = new Hotel
{
    Name = dto.Name,
    City = dto.City,
    Star = dto.Star
};

```

AutoMapper sayesinde bu işlem **tek satıra** düşer:

```csharp
var hotel = _mapper.Map<Hotel>(dto);

```

veya:

```csharp
_mapper.Map(dto, existingHotel);

```

---

## 🧱 Kullanım Örneği

### 1️⃣ Mapping Profile oluştur:

```csharp
using AutoMapper;
using HotelBooking.Domain.Entities;
using HotelBooking.Application.DTOs;

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<CreateHotelDto, Hotel>();
        CreateMap<UpdateHotelDto, Hotel>();
        CreateMap<Hotel, HotelDto>();
    }
}

```

### 2️⃣ Program.cs’de kaydet:

```csharp
builder.Services.AddAutoMapper(typeof(MappingProfile));

```

### 3️⃣ Kullan:

```csharp
var entity = _mapper.Map<Hotel>(createDto);      // DTO → Entity
var dto = _mapper.Map<HotelDto>(entity);         // Entity → DTO
_mapper.Map(updateDto, existingEntity);          // DTO → mevcut Entity'yi güncelle

```

---

## 🔹 AutoMapper’ın Avantajları

|Avantaj|Açıklama|
|---|---|
|⚡ Hızlı geliştirme|Tek tek property atama derdini ortadan kaldırır.|
|🧹 Temiz kod|Servis katmanlarında kod tekrarı azalır.|
|🔄 İki yönlü dönüşüm|DTO ↔ Entity çift yönlü dönüşüm kolay.|
|🔧 Özelleştirilebilir|İsim farkı olan property’leri manuel eşleyebilirsin.|
|🧠 Akıllı eşleştirme|Null, default, nested property gibi durumları yönetebilir.|

---

## 🧠 Özetle

|Kavram|Tanım|Ne işe yarar|
|---|---|---|
|**DTO (Data Transfer Object)**|Katmanlar arası veri taşıyıcı sınıf|Veri taşır, ama iş mantığı içermez|
|**AutoMapper**|Nesneler arası veri dönüştürücü|DTO ↔ Entity dönüşümünü otomatik yapar|

---

## 💬 Kısaca Hatırlat:

> 🔹 DTO: "Veriyi dış dünyadan içeri taşır ya da içeriden dışarı taşır."
> 
> 🔹 AutoMapper: "Bu taşıma işlemini otomatikleştirir."