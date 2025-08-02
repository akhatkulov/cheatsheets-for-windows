https://github.com/lgandx/Responder

### Soxta qurilmamiz orqali amalga oshirilishi mumkin bo‘lgan hujumlar davomida, endi biz tarmoq autentifikatsiyasi protokollariga qarshi hujumlarni ko‘rib chiqamiz.

Windows tarmoqlarida foydalanuvchilarga turli xizmatlardan foydalanish imkonini beradigan juda ko‘p xizmatlar mavjud. Bu xizmatlar kiruvchi ulanishlarning shaxsini tasdiqlash uchun o‘z ichki autentifikatsiya usullaridan foydalanishi kerak.

**2-vazifada** biz veb-ilovada ishlatilgan **NTLM autentifikatsiyasi**ni ko‘rib chiqqan edik. Endi esa, bu autentifikatsiya tarmoq nuqtai nazaridan qanday ko‘rinishda bo‘lishini chuqurroq o‘rganamiz. Bu safar esa, **SMB tomonidan foydalaniladigan NetNTLM autentifikatsiyasiga** e’tibor qaratamiz.

---

### SMB – Server Message Block

**SMB (Server Message Block)** protokoli ishchi stansiyalar (mijozlar) bilan fayl serverlari o‘rtasida aloqani ta’minlaydi. Microsoft Active Directory’dan foydalanuvchi tarmoqlarda SMB fayllarni umumiy ishlatish, masofaviy boshqaruv va hatto printerdan chiqayotgan “qog‘oz tugadi” bildirishnomalarigacha bo‘lgan xizmatlarni boshqaradi.

Ammo, bu protokolning dastlabki versiyalari xavfsiz emas deb topilgan. Ular orqali **credentiallarni (login-parollarni) qo‘lga kiritish** yoki hatto **kod bajarishga** imkon beruvchi zaifliklar aniqlangan. Har doim ham yangi versiyalar joriy qilinmaydi, chunki eski tizimlar ularni qo‘llab-quvvatlamaydi. Biz quyidagi **ikki xil hujum**ni ko‘rib chiqamiz:

1. **NTLM challenge-larni tutib olib**, oflayn tarzda parollarni **crack (buzish)**.
2. Bizning soxta qurilma orqali **Man-in-the-Middle (MitM)** hujum qilib, mijoz va server o‘rtasida autentifikatsiyani **relay** qilish va faol sessiyani egallash.

---

### LLMNR, NBT-NS va WPAD

Bu bosqichda biz **SMB ishlatilganda yuz beradigan autentifikatsiyani** o‘rganamiz. Biz bu yerda **Responder** vositasidan foydalanamiz va **NetNTLM challenge**ni tutib olib, parollarni crack qilishga harakat qilamiz.

Katta Windows tarmoqlarida kompyuterlar LLMNR, NBT-NS va WPAD kabi protokollar orqali o‘zaro aloqada bo‘lishadi. Masalan:

* **LLMNR (Link-Local Multicast Name Resolution)** – lokal tarmoqdagi DNS muammolarini hal qiladi.
* **NBT-NS (NetBIOS Name Service)** – LLMNRdan oldingi protokol.
* **WPAD (Web Proxy Auto-Discovery Protocol)** – avtomatik proksi aniqlash uchun ishlatiladi.

Bu so‘rovlar lokal tarmoq orqali **broadcast** qilinganligi sababli, sizning soxta qurilmangiz ham ularni eshitadi. **Responder** esa bu so‘rovlarni tahlil qilib, **soxta javoblar** yuboradi va qurilmalarni sizning IP manzilingizga yo‘naltiradi.

Shu bilan birga, **SMB, HTTP, SQL va boshqa xizmatlar**ni ko‘rsatib, mijozni sizga bog‘lanishga majbur qiladi. Bu orqali siz **autentifikatsiyani majburan amalga oshirishga** erishasiz.

---

### NetNTLM Challenge-ni tutib olish

Responder – bu holatda “poyga holati”ni yutishga harakat qiladi, ya’ni **soxta javobni tezroq yuborishga** urinish. Siz VPN orqali ulangansiz, shuning uchun faqat VPN tarmog‘idagi autentifikatsiyalarni zaharlash mumkin.

Bu xonada sizga 30 daqiqada bir marta avtomatik ravishda zaharlanishi mumkin bo‘lgan autentifikatsiya so‘rovi **simulyatsiya** qilingan. Shu sababli, Responder-ni **kamida 10 daqiqa ishga tushirib qo‘yish** lozim bo‘ladi.

Misol:

```
[+] Listening for events...
[SMBv2] NTLMv2-SSP Client   : <Client IP>
[SMBv2] NTLMv2-SSP Username : ZA\<Foydalanuvchi nomi>
[SMBv2] NTLMv2-SSP Hash     : <Foydalanuvchi>::ZA:<Hash>
```

Bu hashni `.txt` faylga saqlang. So‘ngra quyidagi buyruq orqali **Hashcat** yordamida parolni topishga harakat qilamiz:

```
hashcat -m 5600 <hash_fayl> <parollar_fayli> --force
```

* `-m 5600` — bu `NTLMv2-SSP` uchun Hashcat rejimi.
* Parollar fayli sizga `/root/Rooms/BreachingAD/task5/` ichida berilgan.

Agar parollar kuchsiz bo‘lsa, siz ularni muvaffaqiyatli crack qilishingiz mumkin va bu orqali sizga kerakli **AD credentiallar** qo‘lga kiritiladi.

---

### Challenge-ni Relay qilish

Yana bir variant — **autentifikatsiyani relay qilish**, ya’ni zaharlangan so‘rovni boshqa serverga uzatish. Buni muvaffaqiyatli qilish uchun quyidagilar kerak:

1. **SMB signing** o‘chirib qo‘yilgan bo‘lishi kerak yoki **yoqilgan ammo majburiy emas** bo‘lishi kerak.
2. **Foydalanuvchi akkaunti** uzatilayotgan resursga kirish ruxsatiga ega bo‘lishi kerak (masalan, admin huquqlariga).

Bu turdagi “blind” relaylar ko‘pincha ishonchsiz, chunki siz hali AD’ni buzmagan bo‘lasiz va foydalanuvchi huquqlari haqida ma’lumotga ega bo‘lmaysiz.

Ammo agar siz ilgari AD’ni buzgan bo‘lsangiz, relay hujum orqali **domain ichida lateral harakat** qilishingiz mumkin.

---

### Yakuniy eslatmalar

* Responder faqat **lokal tarmoq**dagi so‘rovlarga ishlaydi.
* Hujumlar **tarmoqdagi xizmatlarni izdan chiqarishi mumkin**, ehtiyot bo‘ling.
* Hujumni real LANda amalga oshirish – bu ishni yanada samarali qiladi.
* Ko‘proq amaliyot uchun siz **Holo Network**’ga o‘ting yoki keyingi AD xonalarida davom eting.

---

Agar boshqa qismlarini ham tarjima qilishni xohlasangiz, davom ettiraman.
