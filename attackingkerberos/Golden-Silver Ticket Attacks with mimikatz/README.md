**Mimikatz** juda mashhur va kuchli post-ekspluatatsiya vositasi bo'lib, odatda Active Directory tarmog'ida foydalanuvchi hisob ma'lumotlarini olish uchun ishlatiladi. Biroq, biz **kumush chipta** yaratish uchun Mimikatzdan foydalanamiz.

**Kumush chipta** ba'zan **oltin chiptaga** qaraganda operatsiyalarda yaxshiroq ishlatilishi mumkin, chunki u biroz yashirinroq. Agar maxfiylik va sezilmaslik muhim bo'lsa, kumush chipta oltin chiptaga qaraganda yaxshiroq variantdir, ammo ikkalasini yaratish usuli bir xil. Ularning asosiy farqi shundaki, kumush chipta faqat maqsadli servis bilan cheklangan, oltin chipta esa har qanday Kerberos servisiga kirish imkoniyatiga ega.

Kumush chiptaning o'ziga xos ishlatilish misoli: siz domendagi SQL serveriga kirishni istaysiz, ammo hozirgi buzilgan foydalanuvchingiz unga kirish huquqiga ega emas. Siz kerberoasting usuli bilan kirish mumkin bo'lgan servis hisobini topib, undan foydalanishingiz mumkin. Keyin servis xeshini olish (dump qilish) va ularning TGT (Ticket Granting Ticket) ni o'zlashtirish orqali KDC (Key Distribution Center) dan SQL servisi uchun servis chiptasini so'rashingiz mumkin, bu esa sizga domendagi SQL serveriga kirish imkoniyatini beradi.

**KRBTGT haqida tushuncha**

Bu hujumlarning qanday ishlashini to'liq tushunish uchun KRBTGT va TGT orasidagi farqni tushunish kerak. KRBTGT - bu KDC ning servis hisobi bo'lib, u mijozlarga barcha chiptalarni beradi. Agar siz ushbu hisobni o'zlashtirsangiz va KRBTGT dan oltin chipta yaratsangiz, o'zingizga istalgan narsa uchun servis chiptasini yaratish imkoniyatini berasiz. TGT esa KDC tomonidan berilgan servis hisobiga chipta bo'lib, faqat o'sha servisga (masalan, SQLService chiptasi) kirish huquqini beradi.

**Oltin/Kumush Chipta Hujumi - Umumiy ma'lumot**

Oltin chipta hujumi domendagi istalgan foydalanuvchining TGT (ticket-granting ticket) ni olish orqali amalga oshiriladi. Afzalroq variant bu domain admin hisobi bo'lishi kerak, lekin oltin chipta uchun siz **krbtgt** chiptasini, kumush chipta uchun esa har qanday servis yoki domain admin chiptasini olishingiz kerak. Bu sizga servis/domain admin hisobining SID (xavfsizlik identifikatori) va NTLM xeshini taqdim etadi. Keyin siz ushbu tafsilotlarni Mimikatz orqali oltin chipta hujumida berilgan servis hisob ma'lumotlarini o'zlashtirish uchun TGT yaratishda ishlatasiz.

**krbtgt xeshini olish (dump qilish) -**

1. **cd downloads && mimikatz.exe** - Mimikatz joylashgan papkaga o'ting va uni ishga tushiring.
2. **privilege::debug** - Bu [privilege '20' ok] chiqishini tekshiring.
3. **lsadump::lsa /inject /name:krbtgt** - Bu Oltin Chipta yaratish uchun kerak bo'lgan xesh va xavfsizlik identifikatorini chiqaradi. Kumush chipta yaratish uchun **/name:** parametrini o'zgartirib, domain admin hisobi yoki SQLService kabi servis hisobining xeshini olishingiz kerak.

**Oltin/Kumush Chipta yaratish -**

1. **Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:** - Bu oltin chipta yaratish buyrug'i. Kumush chipta yaratish uchun **krbtgt** o'rniga servis NTLM xeshini, **sid** o'rniga servis hisobining SID ni qo'ying va **id** ni 1103 ga o'zgartiring.
   
   Men sizga oltin chipta yaratishni namoyish qilaman, kumush chiptani yaratish sizning vazifangiz.

**Oltin/Kumush Chiptadan boshqa mashinalarga kirish uchun foydalanish -**

1. **misc::cmd** - bu Mimikatzda berilgan chipta bilan yangi yuqori darajadagi (elevated) command prompt ochadi.
2. **Kirishni istagan mashinalaringizga kirishingiz mumkin.** Kirish imkoniyatingiz chiptani olgan foydalanuvchining huquqlariga bog'liq bo'ladi. Agar siz chiptani **krbtgt** dan olgan bo'lsangiz, BUTUN tarmoqqa kirish huquqiga egasiz (shuning uchun "oltin chipta"). Biroq, kumush chiptalar faqat foydalanuvchi kirish huquqiga ega bo'lgan resurslarga kirish imkoniyatiga ega. Agar u domain admin bo'lsa, deyarli butun tarmoqqa kira oladi, ammo oltin chiptaga qaraganda biroz pastroq darajada.
