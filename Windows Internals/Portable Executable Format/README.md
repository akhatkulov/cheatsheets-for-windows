Windows ichki tizimlari yuqori darajada qanday ishlashini belgilab beruvchi asosiy qismlardan biri — **ijro fayllar (executables) va ilovalar** hisoblanadi.

**PE (Portable Executable)** formati ijro fayl va undagi ma’lumotlar haqida umumiy tuzilmani belgilab beradi. Bundan tashqari, PE formati ma’lumot komponentlari qanday saqlanishini ham belgilaydi.

**PE (Portable Executable)** formati — ijro fayllar va obyekt fayllar uchun umumiy tuzilmadir. **PE** va **COFF (Common Object File Format)** fayllari birgalikda PE formatini tashkil etadi.

---

### PE ma’lumotlari

PE ma’lumotlarini eng ko‘p ijro faylining **hex dump** (o‘n oltilik ko‘rinish)ida ko‘rish mumkin. Quyida biz `calc.exe` faylining hex dump’ini tahlil qilib, undagi PE ma’lumotlarini qismlarga ajratib ko‘rsatamiz.

PE ma’lumotlari tuzilishi 7 ta komponentga bo‘linadi:

---

#### 1. **DOS Header**

Fayl turini belgilaydi.

**MZ DOS header** fayl formatini **.exe** sifatida belgilaydi. DOS header quyidagi hex dump qismida ko‘rinib turibdi:

```
00000000  4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00  MZ..........ÿÿ..
00000010  B8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00  ¸.......@.......
```

---

#### 2. **DOS Stub**

Fayl boshida ishga tushadigan kichik dastur bo‘lib, moslik xabarini chiqaradi. Bu oddiy foydalanuvchilar uchun faylning funksionalligiga ta’sir qilmaydi.

DOS stub odatda quyidagi xabarni chiqaradi:
**“This program cannot be run in DOS mode.”**

Hex dump’da:

```
00000050  69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F  is program canno
00000060  74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20  t be run in DOS 
```

---

#### 3. **PE File Header**

Ijro faylining PE header ma’lumotlarini beradi. Fayl formatini, imzo (signature), image file header va boshqa ma’lumotlarni o‘z ichiga oladi.

PE header boshini **PE stub** orqali aniqlash mumkin:

```
000000E0  00 00 00 00 00 00 00 00 50 45 00 00 64 86 06 00  ........PE..d†..
```

---

#### 4. **Image Optional Header**

Nomi "optional" (ixtiyoriy) bo‘lsa ham, bu qism PE File Header’ning muhim qismidir.

**Data Dictionaries** shu qism tarkibida bo‘lib, ular **image data directory structure**ga ko‘rsatkich bo‘lib xizmat qiladi.

---

#### 5. **Section Table**

Image’dagi mavjud bo‘limlar va ularga oid ma’lumotlarni belgilaydi. Bo‘limlar fayl mazmunini saqlaydi, masalan: kod, importlar, ma’lumotlar.

Hex dump’dan misol:

```
000001F0  2E 74 65 78 74 00 00 00 D0 0B 00 00 00 10 00 00  .text...Ð.......
00000210  00 00 00 00 20 00 00 60 2E 72 64 61 74 61 00 00  .... ..`.rdata..
00000240  2E 64 61 74 61 00 00 00 B8 06 00 00 00 30 00 00  .data...¸....0..
00000290  2E 72 73 72 63 00 00 00 10 47 00 00 00 50 00 00  .rsrc....G...P..
```

---

### Asosiy bo‘limlar va ularning vazifalari

| Bo‘lim (Section)        | Vazifasi                                                                |
| ----------------------- | ----------------------------------------------------------------------- |
| **.text**               | Ijro kodi va entry point (boshlanish nuqtasi)ni o‘z ichiga oladi        |
| **.data**               | Initsializatsiya qilingan ma’lumotlar (satrlar, o‘zgaruvchilar va h.k.) |
| **.rdata** / **.idata** | Importlar (Windows API) va DLL’lar                                      |
| **.reloc**              | Relokatsiya ma’lumotlari                                                |
| **.rsrc**               | Ilova resurslari (rasmlar va h.k.)                                      |
| **.debug**              | Debug (nosozliklarni tuzatish) ma’lumotlari                             |

---

👉 Mexroj, xohlaysanmi men senga PE fayl strukturasini oddiy rasm (diagramma) qilib ham chizib beray, shunda yaxshiroq eslab qolasan?
