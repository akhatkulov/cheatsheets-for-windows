Kerberoasting kabi, **AS-REP Roasting** ham Kerberos pre-autentifikatsiyasi o‘chirilgan foydalanuvchi akkauntlarining xeshlarini chiqarib oladi. Kerberoastingdan farqli ravishda, bu hujumda foydalanuvchilar **xizmat (service) akkauntlari bo‘lishi shart emas** — yagona talab shundaki, foydalanuvchi akkauntida **“Kerberos preauthentication talab qilinmasin”** bayrog‘i (UF_DONT_REQUIRE_PREAUTH) yoqilgan bo‘lishi kerak.

Standart Kerberos autentifikatsiyasi jarayonida foydalanuvchining xeshi vaqt belgisi (timestamp)ni shifrlaydi va Key Distribution Center (KDC) ushbu ma’lumotni ochib, foydalanuvchi shaxsini tekshiradi. Ammo agar pre-autentifikatsiya o‘chirilgan bo‘lsa, KDC bu tekshiruv bosqichini o‘tkazib yuboradi va foydalanuvchi shaxsi tasdiqlanmasdan shifrlangan **AS-REP** javobini qaytaradi. Ushbu shifrlangan ma’lumotni ushlab olib, **offline tarzda buzish (cracking)** orqali foydalanuvchi parolini tiklash mumkin.

---

## AS-REP Roasting ikki bosqichda

AS-REP Roasting ikki asosiy bosqichdan iborat: **enumeratsiya** va **ekspluatatsiya**. Avval zaif foydalanuvchi akkauntlari aniqlanadi, so‘ngra olingan AS-REP xeshlar offline tarzda buziladi.

---

## 1-bosqich: Enumeratsiya – Zaif akkauntlarni aniqlash

Bu bosqichda Active Directory ichida **Kerberos pre-autentifikatsiyasi o‘chirilgan** foydalanuvchi akkauntlari aniqlanadi. Pre-autentifikatsiyasiz akkauntlar tarmoqdagi istalgan shaxsga oldindan o‘zini tasdiqlamasdan Kerberos ticket (AS-REP) so‘rash imkonini beradi. Natijada foydalanuvchi parolining shifrlangan xeshi ochiq holda olinadi va offline hujumlarga moyil bo‘ladi.

---

### Ishlatiladigan vositalar

### **Rubeus (faqat Windows)**

Kerberos bilan bog‘liq xavfsizlik testlari va enumeratsiya uchun mo‘ljallangan kuchli Windows vositasi. Rubeus avtomatik ravishda zaif akkauntlarni aniqlaydi va shifrlangan AS-REP xeshlarini oladi.

**Misol buyruq:**

```
Rubeus.exe asreproast
```

Bu buyruq Active Directory’ni skan qiladi, pre-autentifikatsiyasi o‘chirilgan akkauntlarni topadi va offline buzish uchun tayyor xeshlarni chiqaradi.

---

### **Impacket’ning GetNPUsers.py (Linux/Windows)**

Impacket Linux yoki boshqa Windows bo‘lmagan muhitlarda ishlash uchun qulay Python skriptini taqdim etadi. Pre-autentifikatsiya zaifligini tekshirish uchun foydalanuvchi nomlari yozilgan **users.txt** fayli kerak bo‘ladi.

**Misol buyruq:**

```
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

Bu buyruq users.txt ichidagi foydalanuvchilarni tekshiradi va zaif akkauntlarga tegishli AS-REP xeshlarni yig‘ib, ularni **hashes.txt** fayliga saqlaydi.

---

Biz bu yerda **ikkinchi yondashuv**ni tanlaymiz, chunki tarmoqda Windows mashinaga ega emasmiz. Shu sababli AttackBox’dan foydalanamiz.

Vazifa faylini yuklab olish uchun sahifaning yuqorisidagi **Download Task Files** tugmasini bosing. AttackBox’da foydalanuvchi nomlari ro‘yxatini yaratishning eng oson usuli — terminalni ochib, quyidagi buyruqlarni bajarish:

```
cat > users.txt
```

Foydalanuvchi nomlarini joylashtiring va **Ctrl + D** tugmalarini bosing. So‘ng fayl yaratilganini tekshiring:

```
cat users.txt
```

Misol:

```
Administrator
Guest
krbtgt
sshd
gerald.burgess
[...]
```

---

Keyin quyidagi buyruqni ishga tushiramiz:

```
GetNPUsers.py tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
```

Natijada **hashes.txt** fayli yaratiladi.

---

## 2-bosqich: Ekspluatatsiya – Parollarni buzish va tarmoqqa kirish

Enumeratsiya bosqichida olingan shifrlangan xeshlar endi offline tarzda buziladi. Agar parol tiklansa, ushbu foydalanuvchi nomidan autentifikatsiya qilish, Kerberos ticket olish yoki tarmoq resurslariga kirish mumkin bo‘ladi.

---

### Ishlatiladigan vosita

### **Hashcat (ko‘p platformali)**

Hashcat — juda tez va keng tarqalgan parol buzish vositasi. AS-REP xeshlar uchun **18200-rejim** ishlatiladi.

**Misol buyruq:**

```
hashcat -m 18200 hashes.txt wordlist.txt
```

**Buyruq tushuntirishi:**

* `-m 18200` — AS-REP Kerberos xesh rejimi
* `hashes.txt` — yig‘ilgan xeshlar fayli
* `wordlist.txt` — parollar ro‘yxati

Biz bu yerda **rockyou.txt** wordlist’idan foydalanamiz. U `/usr/share/wordlists/rockyou.txt` manzilida joylashgan.

```
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

Bir necha daqiqa ichida parol buziladi. Mashq qiziqarli bo‘lishi uchun natijadagi parol **[PASSWORD REDACTED]** bilan yashirilgan.

---

## Parol buzilgandan so‘ng

Xesh muvaffaqiyatli buzilgach, **ochiq (plaintext) parollar** qo‘lga kiritiladi. Endi ushbu foydalanuvchi sifatida tizimga kirish, Kerberos ticket so‘rash yoki boshqa tarmoq resurslariga murojaat qilish mumkin.

---

## Himoya choralar (Mitigations)

* Barcha foydalanuvchi akkauntlari uchun Kerberos pre-autentifikatsiyasini majburiy qilish
* Kuchli va murakkab parollar offline buzishni sekinlashtiradi
* KDC’da g‘ayrioddiy AS-REP so‘rovlarini monitoring qilish

---

## Asosiy xulosalar

* AS-REP Roasting — shovqinsiz va autentifikatsiyasiz Kerberos hujumi
* Windows’da Rubeus, boshqa tizimlarda GetNPUsers.py qulay
* Hujum muvaffaqiyati parol kuchi va pre-autentifikatsiya sozlamalariga bog‘liq

