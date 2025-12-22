**Mimikatz** - bu faol direktoriya tarmog‘ida foydalanuvchi hisob ma’lumotlarini chiqarish uchun eng mashhur va kuchli post-ekspluatatsiya vositasi. Biz LSASS xotirasidan TGT ni chiqarish uchun Mimikatz dan foydalanamiz.

Bu faqat "pass the ticket" hujumlari qanday ishlashini tushuntirish bo‘lib, THM hozirda tarmoqlarni qo‘llab-quvvatlamaydi, lekin sizni o‘zingizning tarmog‘ingizda sozlashga chaqiraman.

Siz bu hujumni berilgan mashinada ham amalga oshirishingiz mumkin, ammo domain controller sozlamalari tufayli domain admin dan yana domain admin ga o‘tishingiz mumkin.

---

**Pass the Ticket - Umumiy ko‘rinish**

"Pass the ticket" hujumi mashina LSASS xotirasidan TGT ni chiqarish orqali amalga oshiriladi. LSASS (Local Security Authority Subsystem Service) - bu faol direktoriya serverida hisob ma’lumotlarini saqlaydigan xotira jarayoni bo‘lib, u Kerberos bileti va boshqa hisob ma’lumotlarini saqlashi mumkin. Siz hash larni chiqarganingiz kabi Kerberos biletlarini ham LSASS xotirasidan chiqarishingiz mumkin. Mimikatz yordamida biletlarni chiqarganda, bizga .kirbi formatidagi bilet beriladi, agar domain admin bileti LSASS xotirasida bo‘lsa, undan domain admin huquqlarini olish uchun foydalanish mumkin.

Bu hujum privilegiyalarni oshirish va lateral harakatlanish uchun juda yaxshi, agar himoyasiz domain xizmat hisobi biletlari mavjud bo‘lsa. Agar siz domain admin biletini chiqarib olsangiz va Mimikatz PTT hujumi yordamida o‘sha biletni impersonate qilsangiz, siz domain admin sifatida harakat qila olasiz. "Pass the ticket" hujumini mavjud biletni qayta ishlatish deb tushunishingiz mumkin - biz yangi bilet yaratmaymiz yoki yo‘q qilmaymiz, balki domendagi boshqa foydalanuvchining mavjud biletini qayta ishlatamiz va o‘sha biletni impersonate qilamiz.

---

**Mimikatz ni Tayyorlash va Biletlarni Chiqarish**

Administrator sifatida command prompt ishga tushirishingiz kerak: mashinaga kirishda ishlatgan hisob ma’lumotlaringiz bilan.

1.) **cd Downloads** - Mimikatz joylashgan papkaga o‘ting.

2.) **mimikatz.exe** - Mimikatz ni ishga tushiring.

3.) **privilege::debug** - Bu buyruq `[output '20' OK]` chiqarishini tekshiring. Agar chiqmasa, Mimikatz ni to‘g‘ri ishlatish uchun administrator huquqlariga ega emassiz.

4.) **sekurlsa::tickets /export** - bu barcha .kirbi biletlarini joriy papkaga eksport qiladi.

Bu bosqichda biz ilgari Rubeus bilan yig‘ib olgan base64 kodlangan biletlardan ham foydalanishimiz mumkin.

Qaysi biletni impersonate qilishni tanlashda, yuqorida qizil rangda ko‘rsatilgani kabi **krbtgt** dan administrator bileti qidirishni tavsiya qilaman.

---

**Mimikatz yordamida Pass the Ticket**

Endi bilet tayyor bo‘lgani uchun, domain admin huquqlarini olish uchun "pass the ticket" hujumini amalga oshirishimiz mumkin.

1.) **kerberos::ptt <ticket>** - bu buyruqni Mimikatz ichida oldin yig‘ib olgan bilet bilan ishga tushiring. U berilgan biletni keshlaydi va impersonate qiladi.

2.) **klist** - bu orqali keshlangan biletlar ro‘yxatini ko‘rib, biletni muvaffaqiyatli impersonate qilganimizni tasdiqlaymiz.

Hujumning qolgan qismi uchun Mimikatz dan foydalanmaymiz.

3.) Endi siz biletni impersonate qildingiz, bu sizga impersonate qilayotgan TGT bilan bir xil huquqlarni beradi. Buni tekshirish uchun admin share ni ko‘rishimiz mumkin.

Eslatma: Bu faqat "pass the ticket" qanday amalga oshirilishi va domain admin huquqlarini olishni tushunish uchun POC (proof of concept) hisoblanadi. Sizning "pass the ticket" ga yondashuvingiz qanday auditda ekanligingizga qarab farq qilishi mumkin, shuning uchun buni ushbu hujumni amalga oshirishning yakuniy qo‘llanmasi deb olmang.

