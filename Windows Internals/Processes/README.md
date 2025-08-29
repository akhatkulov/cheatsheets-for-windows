
**Jarayon (Process)** – bu dastur bajarilishini ifodalaydi va boshqaradi; bitta ilova bir yoki bir nechta jarayonlardan iborat bo‘lishi mumkin. Jarayon ko‘plab tarkibiy qismlarga ega bo‘lib, ular saqlanishi va ular bilan o‘zaro ishlash uchun ajratiladi. Microsoft hujjatlarida jarayon quyidagicha ta’riflanadi:
*"Har bir jarayon dastur bajarilishi uchun zarur bo‘lgan resurslarni taqdim etadi. Jarayon virtual adres maydoniga, bajariladigan kodga, tizim obyektlariga ochiq tutqichlarga, xavfsizlik kontekstiga, noyob jarayon identifikatoriga, muhit o‘zgaruvchilariga, ustuvorlik sinfiga, minimal va maksimal ishchi to‘plam hajmlariga hamda kamida bitta bajariladigan ip (thread)ga ega bo‘ladi."*

Bu ma’lumotlar biroz murakkab ko‘rinishi mumkin, lekin ushbu bo‘lim bu tushunchani oddiyroq qilib tushuntirishga qaratilgan.

Avval aytib o‘tilganidek, jarayonlar ilovalarni ishga tushirish orqali yaratiladi. Jarayonlar Windows ishlashining markazida turadi – Windows’ning deyarli barcha funksiyalari ilova sifatida ko‘rilishi va unga mos jarayon bo‘lishi mumkin. Quyida Windows’da odatda avtomatik ishga tushadigan ba’zi ilovalar va ularning jarayonlari keltirilgan:

* **MsMpEng** (Microsoft Defender)
* **wininit** (klaviatura va sichqoncha)
* **lsass** (credential storage – parol va autentifikatsiya ma’lumotlari)

Hujumchilar ko‘pincha jarayonlarni nishonga olib, aniqlanishdan qochish yoki zararli dasturlarni qonuniy jarayon sifatida yashirish uchun foydalanadilar. Quyida jarayonlarga qarshi ishlatilishi mumkin bo‘lgan ba’zi hujum vektorlari keltirilgan:

* **Process Injection (T1055)**
* **Process Hollowing (T1055.012)**
* **Process Masquerading (T1055.013)**

---

### Jarayonning asosiy komponentlari

Jarayon ko‘plab qismlardan iborat bo‘lsa-da, uni yuqori darajada tavsiflash uchun asosiy komponentlarga ajratish mumkin. Quyidagi jadval ularni ko‘rsatadi:

| **Jarayon komponenti**        | **Vazifasi**                                                                                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Private Virtual Address Space | Jarayonga ajratilgan virtual xotira adreslari                                                                  |
| Executable Program            | Virtual adres maydonida saqlanadigan kod va ma’lumotlarni belgilaydi                                           |
| Open Handles                  | Jarayon foydalanishi mumkin bo‘lgan tizim resurslari tutqichlari                                               |
| Security Context              | Foydalanuvchi, xavfsizlik guruhlari, imtiyozlar va boshqa xavfsizlik ma’lumotlarini belgilaydigan access token |
| Process ID                    | Jarayonning noyob raqamli identifikatori                                                                       |
| Threads                       | Jarayonda bajarilishga rejalashtirilgan bo‘laklar                                                              |

---

### Jarayonning xotiradagi tarkibi

Jarayonni yanada pastroq darajada – virtual adres maydonida qanday joylashishini ham ko‘rib chiqish mumkin:

| **Komponent**     | **Vazifasi**                                       |
| ----------------- | -------------------------------------------------- |
| Code              | Jarayon tomonidan bajariladigan kod                |
| Global Variables  | Saqlanadigan o‘zgaruvchilar                        |
| Process Heap      | Ma’lumotlar saqlanadigan heap xotira maydoni       |
| Process Resources | Jarayon foydalanadigan qo‘shimcha resurslar        |
| Environment Block | Jarayon haqidagi ma’lumotlarni belgilovchi tuzilma |

---

### Task Manager orqali kuzatish

Jarayonlarni yanada aniqroq ko‘rish uchun **Windows Task Manager** dan foydalanish mumkin. Task Manager jarayon haqida turli ma’lumotlarni ko‘rsatadi. Quyida muhim detallar jadvalda keltirilgan:

| **Qiymat/Komponent** | **Vazifasi**                                                                      | **Misol**   |
| -------------------- | --------------------------------------------------------------------------------- | ----------- |
| Name                 | Jarayon nomi, odatda ilova nomidan olinadi                                        | conhost.exe |
| PID                  | Jarayonni aniqlash uchun noyob raqamli qiymat                                     | 7408        |
| Status               | Jarayonning holatini belgilaydi (running, suspended va hokazo)                    | Running     |
| User name            | Jarayonni ishga tushirgan foydalanuvchi. Jarayon imtiyozlarini ko‘rsatishi mumkin | SYSTEM      |

End-user (oddiy foydalanuvchi) yoki hujumchi sifatida jarayonlar bilan eng ko‘p shu komponentlar orqali ishlaysiz.

---

### Jarayonlarni kuzatish uchun vositalar

Jarayonlarni kuzatishni yengillashtiradigan bir nechta qo‘shimcha vositalar mavjud:

* **Process Hacker 2**
* **Process Explorer**
* **Procmon**

---

**Xulosa:**
Jarayonlar Windows ichki komponentlarining markazida turadi. Keyingi bosqichlarda jarayonlar qanday ishlatilishini va ularni qanday ekspluatatsiya qilish mumkinligini yanada chuqurroq o‘rganamiz.

