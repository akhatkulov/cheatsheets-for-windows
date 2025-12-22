**Mimikatz** - bu faol direktoriya tarmog‘ida foydalanuvchi hisob ma’lumotlarini chiqarish uchun eng mashhur va kuchli post-ekspluatatsiya vositasi, lekin biz undan kumush bilet (silver ticket) yaratish uchun foydalanamiz.

Kumush bilet ba‘zan oltin biletga (golden ticket) qaraganda auditlarda yaxshiroq ishlatilishi mumkin, chunki u biroz kamroq sezilarli. Agar yashirinish va sezilmaslik muhim bo‘lsa, kumush bilet oltin biletga qaraganda yaxshiroq variant bo‘ladi, ammo ularni yaratish usuli bir xil. Ikki bilet o‘rtasidagi asosiy farq shundaki, kumush bilet faqat maqsadli xizmat bilan chegaralangan, oltin bilet esa har qanday Kerberos xizmatiga kirish imkoniyatiga ega.

Kumush bilet uchun aniq bir stsenariy: siz domendagi SQL serveriga kirishni xohlaysiz, ammo hozirgi buzilgan foydalanuvchingiz bu serverga kirish huquqiga ega emas. Siz kirish mumkin bo‘lgan xizmat hisobini topib, uni kerberoasting qilish orqali o‘rnashishingiz mumkin, keyin xizmat hash ini chiqarib, ularning TGT sini impersonate qilib, KDC dan SQL xizmati uchun xizmat bileti so‘rash orqali domendagi SQL serveriga kirish huquqini olishingiz mumkin.

---

**KRBTGT - Umumiy ko‘rinish**

Bu hujumlarning qanday ishlashini to‘liq tushunish uchun KRBTGT va TGT o‘rtasidagi farqni tushunishingiz kerak. KRBTGT - bu KDC (Key Distribution Center) uchun xizmat hisobi bo‘lib, u barcha mijozlarga biletlar beradi. Agar siz ushbu hisobni impersonate qilsangiz va KRBTGT dan oltin bilet yaratmasangiz, o‘zingiz uchun istalgan narsa uchun xizmat bileti yaratish imkoniyatiga ega bo‘lasiz. TGT esa KDC tomonidan berilgan xizmat hisobiga bilet bo‘lib, faqat o‘sha xizmatga kirish huquqini beradi (masalan, SQLService bileti).

---

**Oltin/Kumush Bilet Hujumi - Umumiy ko‘rinish**

Oltin bilet hujumi domendagi har qanday foydalanuvchining ticket-granting ticket (TGT) ini chiqarish orqali amalga oshiriladi. Bu afzalroq domain admin bo‘lishi kerak, ammo oltin bilet uchun siz krbtgt biletini, kumush bilet uchun esa har qanday xizmat yoki domain admin biletini chiqarasiz. Bu sizga xizmat/domain admin hisobining SID (security identifier) va NTLM hash ini beradi. Keyin siz ushbu tafsilotlarni Mimikatz oltin bilet hujumida foydalanib, berilgan xizmat hisobi ma’lumotlarini impersonate qiluvchi TGT yaratishingiz mumkin.

---

**krbtgt hash ini chiqarish**

1.) **cd downloads && mimikatz.exe** - Mimikatz joylashgan papkaga o‘ting va Mimikatz ni ishga tushiring.

2.) **privilege::debug** - `[privilege '20' ok]` chiqishini tekshiring.

3.) **lsadump::lsa /inject /name:krbtgt** - Bu Oltin Bilet yaratish uchun kerak bo‘lgan hash va security identifier ni chiqaradi. Kumush bilet yaratish uchun `/name:` parametrini o‘zgartirib, domain admin hisobi yoki SQLService kabi xizmat hisobining hash ini chiqarishingiz kerak.

---

**Oltin/Kumush Bilet Yaratish**

1.) **Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:** - Bu oltin bilet yaratish buyrug‘i. Kumush bilet yaratish uchun krbtgt o‘rniga xizmat NTLM hash ini, sid o‘rniga xizmat hisobi SID ini qo‘ying va id ni 1103 ga o‘zgartiring.

Men sizga oltin bilet yaratishni namoyish qilaman, kumush bilet yaratish sizning ixtiyoringizda.

---

**Oltin/Kumush Bilet yordamida boshqa mashinalarga kirish**

1.) **misc::cmd** - bu Mimikatz da berilgan bilet bilan yangi elevated command prompt ochadi.

2.) Kirishni xohlagan mashinalaringizga kiring. Nimalarga kirishingiz siz biletni olgan foydalanuvchining huquqlariga bog‘liq bo‘ladi. Agar siz krbtgt dan bilet olgan bo‘lsangiz, BUTUN tarmoqqa kirish huquqiga ega bo‘lasiz (shuning uchun "oltin bilet" deb ataladi). Kumush biletlar esa faqat foydalanuvchi kirish huquqiga ega bo‘lgan narsalarga kirish huquqiga ega. Agar u domain admin bo‘lsa, deyarli butun tarmoqqa kirishi mumkin, ammo oltin biletga qaraganda biroz pastroq privilegiyalarga ega.

Bu hujum domendagi boshqa mashinalarsiz ishlamaydi, lekin sizni o‘zingizning tarmog‘ingizda buni sozlash va ushbu hujumlarni sinab ko‘rishga chaqiraman.
