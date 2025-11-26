
## ❖ Command Prompt

Ba’zan Active Directory bo‘yicha **tez va “quick & dirty”** enum qilish kerak bo‘ladi — mana shu joyda Command Prompt (CMD) yordamga keladi. Ishonchli eski CMD, masalan, sizda RDP kirish bo‘lmasa, Blue Team PowerShell ishlatilishini kuzatayotgan bo‘lsa yoki siz AD enumni **RAT orqali** amalga oshirayotgan bo‘lsangiz, juda qo‘l keladi. Hatto fishing payload ichiga bir nechta AD enum buyruqlarini joylab, yakuniy hujumni rejalashtirish uchun muhim ma’lumotlarni olishda foydali bo‘lishi mumkin.

CMD ichida Active Directory haqida ma’lumot olishga yordam beruvchi **built-in buyruq — `net`** mavjud. `net` buyruği orqali lokal tizim va AD haqida ma’lumotlarni enum qilish mumkin. Quyida ba’zi foydali variantlar ko‘rib chiqiladi, ammo bu to‘liq ro‘yxat emas.

**Eslatma:** Bu topshiriqni bajarish uchun **THMJMP1** mashinasidan foydalaning, o‘z Windows VM'ingiz ishlamaydi — bunga sabablar keyin tushuntiriladi.

---

## 👤 Foydalanuvchilar (Users)

AD domenidagi barcha foydalanuvchilarni ko‘rish uchun:

```
C:\>net user /domain
```

Natija:

```
The request will be processed at a domain controller for domain za.tryhackme.com

User accounts for \\THMDC
-------------------------------------------------------------------------------
aaron.conway             aaron.hancock            aaron.harris
aaron.johnson            aaron.lewis              aaron.moore
aaron.patel              aaron.smith              abbie.joyce
[....]
The command completed successfully.
```

Bu bizga domen hajmi haqida tasavvur beradi va keyingi hujumlarni rejalashtirishda yordam beradi.

Agar bitta foydalanuvchi haqida batafsil ma’lumot kerak bo‘lsa:

```
C:\>net user zoe.marshall /domain
```

Natijada quyidagi kabi ma’lumot chiqadi:

* To‘liq ism
* Parol oxirgi marta qachon o‘zgartirilgani
* Parol muddati
* Account aktivligi
* Qaysi guruhlarga tegishliligi
* Logon vaqtlari va boshqalar

**Eslatma:** Agar foydalanuvchi 10+ guruhga a’zo bo‘lsa, `net user` ularning barchasini ko‘rsata olmaydi.

---

## 👥 Guruhlar (Groups)

Domen guruhlarini ko‘rish uchun:

```
C:\>net group /domain
```

Bu orqali:

* Domain Admins
* Domain Users
* Schema Admins
* Tier 0/1/2 Admins
* Domain Controllers

va boshqa guruhlarni ko‘rish mumkin.

Agar ma’lum guruh a’zolarini bilmoqchi bo‘lsak:

```
C:\>net group "Tier 1 Admins" /domain
```

Natijada guruh ichidagi foydalanuvchilar ro‘yxati chiqadi.

---

## 🔐 Parol siyosati (Password Policy)

Domenning parol siyosatini ko‘rish:

```
C:\>net accounts /domain
```

Bu orqali quyidagilar aniqlanadi:

* Minimal parol uzunligi
* Parol muddati tugashi
* Parol tarixining uzunligi (history)
* Lockout threshold — nechta noto‘g‘ri urinishdan keyin bloklanishi
* Lockout davomiyligi
* Foydalanuvchi parolini o‘zgartirishi mumkinligi

Bu ma’lumot **password spraying** hujumlarini rejalashtirishda juda muhim:

* Qanday parollarni sinash kerak?
* Necha marta sinash xavfsiz?
* Hisob bloklanmasligi uchun ehtiyot choralar

Ammo hali ham ko‘r-ko‘rona spray blokirovkaga olib kelishi mumkin.

---

## ✅ Afzalliklar

* Hech qanday qo‘shimcha vosita kerak emas — Windows ichida mavjud
* Ko‘pincha Blue Team bu buyruqlarni log qilmaydi
* GUI talab qilinmaydi
* VBScript, makrolar yoki phishing payloadlarda ishlatish oson
* Minimal “noiz” bilan ADdan foydali ma’lumot olish mumkin

---

## ⚠️ Kamchiliklar

* Buyruqlar **faqat domen-join bo‘lgan mashinada** ishlaydi
* `WORKGROUP` bo‘lsa, AD ma’lumot chiqmaydi
* 10 dan ko‘p guruh a’zoligi bo‘lsa, to‘liq ko‘rsatmaydi

