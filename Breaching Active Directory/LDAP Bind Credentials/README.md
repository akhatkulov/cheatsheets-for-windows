**LDAP**

Ilovalarda Active Directory (AD) autentifikatsiyasi uchun yana bir usul bu **LDAP (Lightweight Directory Access Protocol)** autentifikatsiyasidir. LDAP autentifikatsiyasi NTLM autentifikatsiyasiga o‘xshaydi. Biroq, LDAP autentifikatsiyasida ilova foydalanuvchi ma’lumotlarini bevosita tekshiradi. Ilovaning o‘zida LDAP orqali so‘rov yuborish va keyin AD foydalanuvchisini tekshirish uchun bir juft AD hisob ma’lumotlari bo‘ladi.

LDAP autentifikatsiyasi odatda uchinchi tomon (ya’ni Microsoft bo‘lmagan) ilovalari tomonidan AD bilan integratsiya qilishda keng qo‘llaniladi. Bunday ilova va tizimlarga quyidagilar kiradi:

* Gitlab
* Jenkins
* Maxsus ishlab chiqilgan veb-ilovalar
* Printerlar
* VPNlar

Agar bu ilova yoki xizmatlardan birortasi internetga ochiq bo‘lsa, NTLM asosidagi tizimlarga qarshi qo‘llaniladigan hujumlar bu yerda ham amalga oshirilishi mumkin. Biroq, LDAP autentifikatsiyasi AD bilan ishlash uchun xizmatga AD hisob ma’lumotlari kerak bo‘lganligi sababli, bu holat qo‘shimcha hujum imkoniyatlarini yaratadi. Ya’ni, xizmatda ishlatiladigan AD hisob ma’lumotlarini qo‘lga kiritish orqali AD tizimiga autentifikatsiyalangan kirish imkoniyatini qo‘lga kiritish mumkin bo‘ladi.

Quyida LDAP orqali autentifikatsiya jarayoni ko‘rsatilgan:
*(rasmga ishora qilinmoqda)*

Agar to‘g‘ri serverga kirish imkoniyatiga ega bo‘lsak, masalan Gitlab serveri, unda ushbu AD hisob ma’lumotlarini tiklash ba’zida oddiygina konfiguratsiya fayllarini o‘qish orqali amalga oshiriladi. Bu hisob ma’lumotlari ko‘pincha konfiguratsiya fayllarida oddiy matn ko‘rinishida saqlanadi, chunki xavfsizlik modeli ularni yashirish o‘rniga server xavfsizligiga tayanadi.

![LDAP](https://github.com/akhatkulov/cheatsheets-for-windows/blob/main/Breaching%20Active%20Directory/LDAP%20Bind%20Credentials/d2f78ae2b44ef76453a80144dac86b4e.png?raw=true)
