**JSON (JavaScript Object Notation)** — verileri **hafif**, **okunabilir** ve **makineler arası aktarılabilir** bir formatta tutan bir yapıdır.

Genellikle **API’ler arasında veri alışverişi** için kullanılır.

---

### ✅ JSON Örneği:

```json
{
  "Id": 1,
  "FirstName": "Ali",
  "LastName": "Yılmaz",
  "Age": 30,
  "City": "Istanbul"
}

```

### 💡 Özellikleri:

- Anahtar–değer (key–value) mantığıyla çalışır.
- `"` çift tırnak zorunludur.
- Veri tipleri: string, number, boolean, array, object.
- **Modern web API’lerde en çok kullanılan formattır.**
- İnsan tarafından okunabilir.

## 🧩 XML Nedir?

**XML (eXtensible Markup Language)** — verileri **etiket (tag)** yapısıyla tanımlayan bir formattır.

Eskiden çok yaygındı, halen bazı eski sistemlerde ve konfigürasyon dosyalarında kullanılır.

---

### ✅ XML Örneği:

```xml
<Employee>
  <Id>1</Id>
  <FirstName>Ali</FirstName>
  <LastName>Yılmaz</LastName>
  <Age>30</Age>
  <City>Istanbul</City>
</Employee>

```

### 💡 Özellikleri:

- HTML benzeri etiket yapısı kullanır.
- Açılış ve kapanış etiketleri zorunludur.
- Daha **ağır** ama **daha esnek** bir formattır.
- Eski SOAP tabanlı web servislerde yaygındı.