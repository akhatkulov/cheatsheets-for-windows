### Group Policy Preferences (GPP)

Windows operatsion tizimida **Administrator** nomli o‘rnatilgan hisob mavjud bo‘lib, unga parol orqali kirish mumkin. Ko‘plab kompyuterlar mavjud bo‘lgan katta Windows muhitida ushbu mahalliy administrator hisoblari parollarini o‘zgartirish juda murakkab jarayondir. Shu sababli, Microsoft butun domen bo‘yicha mahalliy administrator hisoblari parollarini boshqarish uchun **Group Policy Preferences (GPP)** deb nomlangan metodni joriy qilgan.

**GPP** — bu administratorlarga domen siyosatlarini parollarni o‘z ichiga olgan holda yaratishga imkon beruvchi vosita. Bunday siyosatlar tatbiq etilgach, turli **XML** fayllar **SYSVOL** papkasida yaratiladi. **SYSVOL** — bu Active Directory’ning muhim komponenti bo‘lib, NTFS disk bo‘limida barcha autentifikatsiyadan o‘tgan domen foydalanuvchilari tomonidan **o‘qish huquqi bilan** ochiq bo‘lgan umumiy papkani yaratadi.

Muammo shundaki, **GPP bilan bog‘liq XML fayllarida** parollar **AES-256** bitli shifrlash bilan shifrlangan holda saqlanardi. O‘sha paytda bu shifrlash yetarli darajada xavfsiz deb hisoblangan, biroq **Microsoft tasodifan shaxsiy kalitini (private key)** MSDN’da e’lon qilib yuborgan. Natijada, domen foydalanuvchilari SYSVOL tarkibini o‘qiy olishlari sababli, saqlangan parollarni osongina **dekodlash** (shifrdan yechish) imkoniyati paydo bo‘lgan. SYSVOLdagi shifrlangan parollarni ochish uchun **Get-GPPPassword** kabi vositalar mavjud.

---

### Local Administrator Password Solution (LAPS)

2015-yilda Microsoft parollarni SYSVOL papkasida saqlashni bekor qildi. Buning o‘rniga u **Local Administrator Password Solution (LAPS)** deb nomlangan xavfsizroq usulni joriy etdi.

Bu yangi usul Active Directory’da har bir kompyuter obyektiga ikkita yangi atribut qo‘shishni o‘z ichiga oladi:

* `ms-mcs-AdmPwd` – mahalliy administrator hisobining **aniq matn ko‘rinishidagi (clear-text)** parolini saqlaydi.
* `ms-mcs-AdmPwdExpirationTime` – ushbu parol qachon **muddatini tugatib**, yangilanishi kerakligini bildiradi.

**LAPS**, kompyuterlarning mahalliy administrator parollarini avtomatik yangilab boradi va yuqoridagi atributlar qiymatini yangilash uchun `admpwd.dll` dan foydalanadi.

![]()
