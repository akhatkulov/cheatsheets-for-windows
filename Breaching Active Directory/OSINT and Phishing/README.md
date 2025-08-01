## 🔍 **OSINT (Open Source Intelligence)** — Ochiq manbalardan razvedka

**OSINT** — bu internetda **ommaviy ravishda mavjud** bo‘lgan ma’lumotlarni izlab topish jarayonidir. AD tizimi uchun foydalanuvchi login va parollari ba’zida **tasodifan yoki bilmasdan** internetga chiqib ketgan bo‘ladi. Misollar:

1. 👨‍💻 **Foydalanuvchilar** Stack Overflow kabi forumlarda savol so‘rab, **login yoki parollarini ochiqlab qo‘yishi**.
2. 👨‍💻 **Dasturchilar** GitHub yoki boshqa kod hosting saytlariga **parollarni kodga qistirib (hardcoded)** yuklab qo‘yishadi.
3. 🔓 **Ilgari yuz bergan ma’lumot sizishlar** orqali. Masalan, xodimlar ishdagi email bilan boshqa saytlarda ro‘yxatdan o‘tgan bo‘lsa va u sayt hack qilingan bo‘lsa, parollari internetda tarqalgan bo‘lishi mumkin.

### 🛠️ Tekshirish vositalari:

* [HaveIBeenPwned.com](https://haveibeenpwned.com) — emailingiz qandaydir sizishlarda qatnashganmi, tekshiradi.
* [DeHashed.com](https://www.dehashed.com) — email, username yoki parol bo‘yicha global sizishlar bazasini tekshiradi.

📌 **Eslatma:** Topilgan ma’lumotlar **eski bo‘lishi mumkin**, shuning uchun ularni **hozirda ham ishlaydimi yoki yo‘q** – alohida tekshirish zarur.

➡️ Bunday tekshiruvni qanday qilish haqida 3-topshiriqda – **NTLM Authenticated Services** bo‘limida – to‘liqroq o‘rganasiz.

---

## 🎣 **Phishing (Fishing - “qarmoq tashlash”)**

Phishing — foydalanuvchini aldab, uni **login-parolini yozdirib olish** yoki **orqa fon (background)** da qandaydir zararli dastur (masalan, RAT – Remote Access Trojan) ni ishga tushirishga majburlash texnikasidir.

### 📋 Phishingning ikki turi ko‘p uchraydi:

1. 🎯 Foydalanuvchini **soxta saytga olib borish**, u yerda u o‘zi login va parolni kiritsa — bu ma’lumotlar hujumchiga yuboriladi.
2. 🖥️ Foydalanuvchiga **"faylni oching" yoki "bu dastur zarur"** deb aytib, aslida esa **RAT** joylashtirish.

RAT ishlaganidan so‘ng, siz **shu foydalanuvchining nomidan tarmoqda harakatlana olasiz**, chunki bu ilova uning kontekstida ishlaydi.

➡️ Shu sababli, Phishing **Red Team** (hujumchilar) va **Blue Team** (himoyachilar) uchun juda **muhim mavzu**.

---

## 📘 Qo‘shimcha o‘rganish:

* **Red Team OSINT** bo‘yicha batafsil xona mavjud (TryHackMe’da).
* **Phishing** bo‘yicha ham alohida maxsus xona mavjud.
