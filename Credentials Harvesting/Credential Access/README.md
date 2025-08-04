Albatta! Quyida matnni **o‘zbek tiliga tarjimasi** va **tushuntirishi** keltirilgan:

---

## 🎯 Credential Access – Kirish Ma’lumotlarini Qo‘lga Kiritish

**Credential access** – bu hujumchilarning tizimda saqlanayotgan foydalanuvchi **parollari** yoki **autentifikatsiya ma’lumotlarini** aniqlashi va qo‘lga kiritishidir. Ular bu orqali boshqa foydalanuvchi sifatida harakat qilib, **lateral movement** (tarmoq bo‘ylab harakatlanish) yoki boshqa resurslarga (dastur, tizim) kirish imkoniyatiga ega bo‘ladi.

Hujumchilar uchun **legitim (haqiqiy) foydalanuvchi ma’lumotlarini** olish tizimdagi zaifliklardan foydalanishdan ko‘ra samaraliroq bo‘ladi.

🧠 Ko‘proq ma’lumot uchun MITRE ATT\&CK’dagi [TA0006](https://attack.mitre.org/tactics/TA0006/) sahifasiga murojaat qilishingiz mumkin.

---

## ❌ Kirish ma’lumotlari (parollar) quyidagi joylarda xavfsiz bo‘lmagan holda saqlanishi mumkin:

* 🔓 Ochiq matnli fayllar
* 🗃️ Ma’lumotlar bazasi fayllari
* 💾 Operativ xotira (RAM)
* 🔐 Parol menejerlari
* 🏛️ Korporativ vaultlar (secur vault)
* 🏢 Active Directory
* 🌐 Tarmoq sniffingi

Keling, har birini alohida ko‘rib chiqamiz:

---

### 📁 Ochiq matnli fayllar

Hujumchilar mahalliy yoki tarmoqqa ulangan fayllar orqali **parollar, maxfiy kalitlar** saqlangan fayllarni izlashadi.

🛠️ MITRE ATT\&CK’dagi nomi: `Unsecured Credentials: Credentials In Files (T1552.001)`

#### Ushbu fayllar qiziqish uyg‘otadi:

* Buyruqlar tarixi (`.bash_history`, PowerShell tarixi)
* Konfiguratsiya fayllari (FTP, Web App, .env)
* Brauzerlar yoki Email ilovalar fayllari
* Zaxira (backup) fayllar
* Tarmoqli umumiy fayllar
* Windows registry
* Source code fayllari

📍 **Masalan:** PowerShell tarixi fayli quyidagi yo‘lda saqlanadi:

```
C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

🧪 Registry’dan “password” so‘zini izlash:

```
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

---

### 🗃️ Ma’lumotlar Bazasi (Database) Fayllari

Dasturlar sozlamalar va parollarni **lokal database fayllarida** saqlaydi. Bu fayllar parollarni olish uchun yaxshi manba bo‘lishi mumkin.

📌 Masalan: **McAfee Endpoint database** faylidan parol ajratib olish – bu [TryHackMe Breaching AD](https://tryhackme.com/room/breachingad) xonasida ko‘rsatilgan.

---

### 🔐 Parol Menejerlari

Parol menejerlari foydalanuvchi loginlarini xavfsiz saqlashga mo‘ljallangan. Ammo **noto‘g‘ri sozlashlar yoki ekspluatatsiya qilinadigan xatoliklar** bu ma’lumotlarni olishga imkon beradi.

🛠️ Misollar:

* Windows Credential Manager
* KeePass
* 1Password
* LastPass

🧰 Brauzerlar yoki lokal ilovalarda saqlangan credential’larni olish uchun turli vositalar mavjud.

---

### 🧠 Xotira Dump (Memory Dump)

Operativ xotirada Windows va boshqa ilovalar ishga tushganida **maxfiy ma’lumotlar** saqlanadi. Xotiraga faqat **admin darajasidagi hujumchilar** kira oladi.

🧾 Xotirada topilishi mumkin:

* Ochiq matnli parollar
* Keshlangan parollar
* AD Kerberos ticket’lar (TGT, TGS)

📍 Bu darsda xotiraga qanday kirish va undan credential’larni qanday olish muhokama qilinadi.

---

### 🧱 Active Directory (AD)

Active Directory – bu katta hajmdagi foydalanuvchi, guruh, kompyuter haqidagi ma’lumotlarni boshqaradi. Shu sababli AD hujumchilarning asosiy nishonidir.

AD quyidagi misconfigurations orqali credential’larni sizdirishi mumkin:

* 📝 **Description maydoni**ga parol yozib qo‘yish
* 📁 **Group Policy SYSVOL**’da kalitlar sizib ketishi
* 🧩 **NTDS.dit** fayli – foydalanuvchi credential’lari
* 🔓 **AD misconfigurations** – hujumga ochiq joylar

---

### 🌐 Tarmoq Sniffingi

Tarmoqqa kirgan hujumchi **Man-In-The-Middle (MITM)** hujumlari orqali **NTLM hash**, parollarni sniff qilishi mumkin.

⚠️ Spoofing orqali AD serveri, DHCP, DNS kabi xizmatlarni soxtalashtirib, foydalanuvchilarni ularga ulanayotganiga ishontirib credential’larni o‘g‘irlaydi.

---

## 🧠 Xulosa

Credential access – bu **eng muhim bosqichlardan biri**, chunki bu orqali hujumchi tizimda to‘liq nazoratga ega bo‘ladi. Har bir bosqichda ehtiyot choralarini ko‘rish, credential’larni shifrlab saqlash, va kerakli monitoring qilish muhim hisoblanadi.

Agar xohlasangiz, har bir bo‘limni amaliy misollar va TryHackMe topshiriqlari bilan davom ettirishingiz mumkin. Tayyor bo‘lsangiz, navbatdagi bosqichga o‘tamizmi?
