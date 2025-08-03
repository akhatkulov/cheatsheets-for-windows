**Lateral Movement (Yon harakatlar) nima?**

Oddiy qilib aytganda, **lateral movement** — bu hujumchilarning tarmoq bo‘ylab harakatlanishida foydalanadigan texnikalar to‘plamidir. Hujumchi tarmoqqa birinchi marta kirgandan so‘ng, quyidagi sabablar tufayli boshqa qurilmalarga o‘tish zarur bo‘ladi:

* Hujumchi sifatida maqsadlarimizga erishish
* Tarmoqqa o‘rnatilgan cheklovlarni chetlab o‘tish
* Tarmoqda qo‘shimcha kirish nuqtalarini yaratish
* Chalg‘itish va aniqlanishdan qochish

Ko‘plab **cyber kill chain** (kiber hujum zanjiri) modellari lateral movementni ketma-ketlikdagi qo‘shimcha bosqich sifatida ko‘rsatadi, ammo aslida bu **takrorlanuvchi sikl** hisoblanadi. Ushbu siklda mavjud bo‘lgan har qanday **credential** (kirish ma’lumotlari) yordamida yangi qurilmalarga kirib boramiz, u yerda esa **privilege escalation** (huquqlarni oshirish) va credentiallarni olishga harakat qilamiz. Yangi credentiallar bilan sikl yana boshidan boshlanadi.

Odatda, ushbu siklni bir necha bor takrorlashga to‘g‘ri keladi, ayniqsa birinchi kirgan qurilmamiz tarmoqdagi resurslarga yetarli huquqqa ega bo‘lmasa.

---

### **Oddiy Misol**

Tasavvur qiling, biz ichki kodlar omboriga kirishimiz kerak bo‘lgan **red team** hujumini bajaryapmiz. Phishing kampaniyasi orqali tarmoqqa birinchi marta kirganmiz. Bunday kampaniyalar odatda texnik bo‘lmagan xodimlarga nisbatan samaraliroq bo‘lganligi uchun biz Marketing bo‘limidagi kompyuter orqali kirganmiz.

Marketing ish stansiyalari, odatda, tarmoqdagi muhim xizmatlarga (masalan, ma’lumotlar bazasi, monitoring xizmatlari yoki kod omborlari) kirishni cheklaydigan firewall siyosatlariga ega bo‘ladi.

Shuning uchun, biz **yadro resurslarga** yetib borish uchun boshqa kompyuterlarga o‘tishimiz (lateral movement) kerak bo‘ladi. Masalan, Marketing kompyuterida administrator huquqlarini qo‘lga kiritamiz va parol **hash**larini chiqaramiz. Agar umumiy lokal administrator hisobini topsak, u boshqa qurilmalarda ham mavjud bo‘lishi mumkin.

Rekonstruksiya davomida biz `DEV-001-PC` nomli qurilmani topamiz va lokal administratorning hashidan foydalangan holda unga ulanib, bu developerga tegishli kompyuter ekanligini aniqlaymiz. U yerdan kod omboriga kirishimiz mumkin.

---

### **Oddiy Lateral Movement**

Lateral movement faqatgina firewall cheklovlarini chetlab o‘tish uchun emas, **aniqlanishdan qochish** uchun ham foydalidir. Hatto Marketing kompyuteri to‘g‘ridan-to‘g‘ri kod omboriga kira olsa ham, biz developerning kompyuteri orqali ulanishni afzal ko‘rgan bo‘lardik — chunki bu **auditor loglarda** kamroq shubhali ko‘rinadi.

---

### **Hujumchi Nigohi Bilan**

Hujumchi sifatida lateral movementni amalga oshirishning bir necha usullari mavjud:

* **Standart administrator protokollari**: WinRM, RDP, VNC yoki SSH orqali boshqa qurilmalarga ulaniladi.
* Foydalanuvchi hisoblari va ular ulanishi mumkin bo‘lgan qurilmalar o‘rtasidagi **mantiqiy aloqani saqlash** zarur. Masalan, IT bo‘limi xodimi veb-serverga ulanayotgan bo‘lsa — bu tabiiy; lekin Marketing kompyuteridan DEV kompyuteriga lokal admin orqali ulanish shubhali ko‘rinadi.

Zamonaviy hujumchilar **aniqlanishni qiyinlashtiradigan** boshqa lateral movement usullaridan ham foydalanishadi. Har qanday texnikani aniqlab bo‘lmaydi deb bo‘lmasa-da, iloji boricha **jim** bo‘lish maqsadga muvofiq.

---

### **Administratorlar va UAC**

Ko‘pgina lateral movement texnikalarida **administrator huquqlari** talab etiladi. Biroq barcha administratorlar teng darajadagi huquqlarga ega emas. Bu yerda farq qilinadigan ikki turdagi administrator mavjud:

1. **Lokal administratorlar** — kompyuterning o‘zida yaratilgan.
2. **Domen administratorlari** — domen orqali lokal administrator guruhiga kiritilgan.

**UAC (User Account Control)** xavfsizlik funksiyasi lokal administratorlar uchun cheklov qo‘yadi. Default `Administrator` hisobidan tashqari, boshqa lokal administratorlar masofadan (RPC, SMB, WinRM) tizimga ulanib, administratorlik vazifalarini bajara olmaydi. Chunki ular **filtered medium token** bilan tizimga kirishadi va bu ularni to‘liq huquqdan mahrum qiladi.

Ammo **domen administratorlari** bu cheklovga duch kelmaydi va to‘liq huquq bilan tizimga kirishadi.

Ba’zi holatlarda ushbu xavfsizlik funksiyasi o‘chirilgan bo‘lishi mumkin, ammo lateral movement muvaffaqiyatsiz tugasa, sabab — **UAC chekloviga duch kelgan lokal administrator** bo‘lishi mumkinligini hisobga olish zarur.
