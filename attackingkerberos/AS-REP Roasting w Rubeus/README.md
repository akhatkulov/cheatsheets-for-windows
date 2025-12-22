Kerberoasting ga juda o‘xshash, **AS-REP Roasting** usuli Kerberos oldindan autentifikatsiyasi (pre-authentication) o‘chirilgan foydalanuvchi hisoblarining krbasrep5 hash larini chiqaradi. Kerberoasting dan farqli o‘laroq, bu foydalanuvchilar xizmat hisoblari bo‘lishi shart emas, AS-REP Roasting qilish uchun yagona talab - foydalanuvchining oldindan autentifikatsiyasi o‘chirilgan bo‘lishi kerak.

Biz Kerberoasting va yig‘ishda (harvesting) bo‘lgani kabi **Rubeus** dan foydalanishni davom ettiramiz, chunki Rubeus AS-REP Roasting qilish va Kerberos oldindan autentifikatsiyasi o‘chirilgan foydalanuvchilarga hujum qilish uchun oddiy va tushunarli buyruqqa ega. Rubeus dan hash ni chiqarib olgandan so‘ng, krbasrep5 hash ni buzish uchun hashcat dan foydalanamiz.

AS-REP Roasting uchun kekeo va Impacket ning GetNPUsers.py kabi boshqa vositalar ham mavjud. Rubeus dan foydalanish osonroq, chunki u avtomatik ravishda AS-REP Roasting qilinishi mumkin bo‘lgan foydalanuvchilarni topadi, GetNPUsers bilan esa avval foydalanuvchilarni aniqlab, qaysilar AS-REP Roasting qilinishi mumkinligini bilishingiz kerak.

Rubeus allaqachon kompilyatsiya qilingan va mashinaga joylashtirilgan.

---

**AS-REP Roasting - Umumiy ko‘rinish**

Oldindan autentifikatsiya jarayonida foydalanuvchi hash i vaqt belgisini (timestamp) shifrlash uchun ishlatiladi, domain controller esa uni deshifrlab, to‘g‘ri hash ishlatilayotganligini va oldingi so‘rov takrorlanmayotganligini tekshiradi. Vaqt belgisi tekshirilgandan so‘ng, KDC foydalanuvchi uchun TGT (Ticket Granting Ticket) beradi. Agar oldindan autentifikatsiya o‘chirilgan bo‘lsa, siz har qanday foydalanuvchi uchun autentifikatsiya ma’lumotlarini so‘rashingiz mumkin va KDC shifrlangan TGT ni qaytaradi, uni oflayn tarzda buzish mumkin, chunki KDC foydalanuvchi haqiqatan kimligini tekshirish qadamini o‘tkazib yuboradi.

---

**Rubeus yordamida KRBASREP5 Hash larini chiqarish**

1.) **cd Downloads** - Rubeus joylashgan papkaga o‘ting.

2.) **Rubeus.exe asreproast** - Bu buyruq zaif (vulnerable) foydalanuvchilarni qidiradi va topilgan zaif foydalanuvchilarning hash larini chiqaradi.

---

**Hashcat yordamida hash larni buzish**

1.) Hash ni nishon mashinasidan hujum mashinangizga o‘tkazing va .txt fayliga joylang.

2.) Birinchi qator **$krb5asrep$23$User...** ko‘rinishida bo‘lishi uchun **$krb5asrep$** dan keyin **23$** qo‘shing.

4-topshiriqda yuklab olgan parollar ro‘yxatidan foydalaning.

3.) **hashcat -m 18200 hash.txt Pass.txt** - Hash larni bazing! Rubeus AS-REP Roasting hashcat ning 18200 rejimidan foydalanadi.

---

**AS-REP Roasting dan Himoyalanish Choralari**

- **Kuchli parol siyosatiga ega bo‘ling**. Kuchli parol bilan hash larni buzish ko‘proq vaqt talab qiladi, bu esa hujumning samaradorligini kamaytiradi.
- **Kerberos oldindan autentifikatsiyasini zarurat bo‘lmaganda o‘chirmang**. Oldindan autentifikatsiyani yoqib qo‘yishdan tashqari ushbu hujumni butunlay oldini olishning boshqa deyarli yo‘li yo‘q.
