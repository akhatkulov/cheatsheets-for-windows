Quyida matnning **o‘zbek tiliga tarjimasi** keltirilgan. Texnik atamalar va buyruqlar asl holicha saqlandi.

---

Bizga endigina Active Directory tarmog‘iga VPN orqali kirish huquqi berildi. Hozircha hech qanday login ma’lumotlarimiz yo‘q; faqat hujumchi mashinamiz eng so‘nggi va kuchli vositalar bilan jihozlangan. Bizning maqsadimiz — Active Directory muhitining tuzilmasini aniqlash, xostlar va servislarni topish hamda tarmoq xaritasini tuzish.

Bir lahzaga tasavvur qilaylik, bizda tayyor tarmoq diagrammasi yo‘q va test doirasi (scope) sifatida quyidagi subnet berilgan:

**10.211.11.0/24**

---

## Host Discovery (Xostlarni aniqlash)

Birinchi qiladigan ishlarimizdan biri — subnet bo‘ylab host discovery skanini ishga tushirishdir. Bu bizga maqsadli tarmoq diapazonida qaysi xostlar faol (live) ekanini aniqlash imkonini beradi. Ko‘pchilik mijozlar pentest scope’iga butun subnetni kiritadi, shuning uchun biz hujum qilmoqchi bo‘lgan barcha faol xostlarni aniqlashimiz kerak.

Biz dastlabki host discovery uchun ikki xil vositani ko‘rsatamiz.

---

## fping

Oddiy `ping` kabi, `fping` ham Internet Control Message Protocol (ICMP) so‘rovlaridan foydalanib, xost tirik yoki yo‘qligini aniqlaydi. Biroq `fping` yordamida subnet kabi bir nechta targetlarni birdan ko‘rsatish mumkin, bu esa uni `ping`ga nisbatan ancha qulay qiladi.

`ping` bitta targetga javob kelguncha yoki timeout bo‘lguncha paket yuborsa, `fping` har bir so‘rovdan keyin keyingi targetga o‘tadi.

Quyidagi buyruq yordamida maqsadli tarmog‘imizdagi faol xostlarni aniqlashimiz mumkin:

### Terminal

```bash
user@tryhackme$ fping -agq 10.211.11.0/24
10.211.11.1
10.211.11.10
10.211.11.20
10.211.11.250
```

**Kalitlar izohi:**

* `-a`: tirik (alive) tizimlarni ko‘rsatadi
* `-g`: berilgan IP/netmask asosida targetlar ro‘yxatini yaratadi
* `-q`: jim rejim, har bir probe natijasini yoki ICMP xatolarini ko‘rsatmaydi

Ushbu buyruqdan so‘ng biz to‘rtta faol xostni aniqladik.
`10.200.12.1` — gateway, `10.200.12.250` esa VPN server, shuning uchun ularni scope’dan tashqarida deb hisoblab, e’tiborga olmaymiz.

Aniqlangan ikkita IP manzilni keyingi port skanlari uchun `hosts.txt` nomli faylga qo‘shishimiz mumkin.

### Misol (Terminal)

```bash
user@tryhackme$ cat hosts.txt
10.211.11.20
10.211.11.10
```

---

## Nmap

Shuningdek, Nmap’ni ping scan rejimida (`-sn`) ishlatib, butun subnetni tekshirishimiz mumkin:

```bash
nmap -sn 10.211.11.0/24
```

* `-sn`: port skan qilmasdan, faqat qaysi xostlar faol ekanini aniqlaydi

---

## Port Scanning (Portlarni skan qilish)

Faol xostlarni aniqlaganimizdan so‘ng, qaysi biri **Domain Controller (DC)** ekanini topishimiz kerak. Bu Active Directory’ga oid muhim servislar qaysilar ekanini va qaysilaridan ekspluatatsiya qilish mumkinligini aniqlashga yordam beradi.

Quyida Active Directory uchun eng ko‘p uchraydigan portlar va protokollar keltirilgan:

| Port | Protokol           | Ma’nosi                                      |
| ---- | ------------------ | -------------------------------------------- |
| 88   | Kerberos           | Kerberos asosidagi enumeratsiya imkoniyati   |
| 135  | MS-RPC             | RPC enumeratsiyasi (null sessionlar)         |
| 139  | SMB/NetBIOS        | Eski SMB ulanishlari                         |
| 389  | LDAP               | AD’ga LDAP so‘rovlar                         |
| 445  | SMB                | Zamonaviy SMB, enumeratsiya uchun juda muhim |
| 464  | Kerberos (kpasswd) | Kerberos parol servisiga oid                 |

DC’ni aniqlash uchun ushbu portlar bo‘yicha servis va versiya skanini ishga tushirishimiz mumkin:

```bash
nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt
```

**Kalitlar izohi:**

* `-sV`: servis versiyalarini aniqlaydi
* `-sC`: Nmap Scripting Engine (NSE) default skriptlarini ishga tushiradi
* `-iL`: target IP’larni `hosts.txt` faylidan o‘qiydi

Odatda Domain Controller’da 88 (Kerberos), 389 (LDAP) va 445 (SMB) portlari ochiq bo‘ladi va bannerlarda `Windows Server` yoki hatto domen nomlari ko‘rinishi mumkin.

---

Agar biz yanada keng qamrovli baholash qilayotgan bo‘lsak yoki notanish muhit bilan ishlayotgan bo‘lsak, barcha portlarni to‘liq skan qilish muhim servislarni standart bo‘lmagan portlarda ham aniqlash imkonini beradi.

Buning uchun quyidagi buyruqdan foydalanish mumkin:

```bash
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

**Kalitlar izohi:**

* `-sS`: TCP SYN scan, full connect scan’ga qaraganda yashirinroq
* `-p-`: barcha 65,535 TCP portni skan qiladi
* `-T3`: tezlik va yashirinlik orasida muvozanat
* `-iL hosts.txt`: oldin aniqlangan xostlar ro‘yxati
* `-oN full_port_scan.txt`: natijalarni faylga saqlaydi

---

Ushbu enumeratsiya usullari orqali biz maqsadli tarmoqda ikkita faol xostni aniqladik:

* bitta **Domain Controller**,
* bitta **Workstation**,
  shuningdek domen nomini ham bilib oldik.

Bundan tashqari, DC’da ishlayotgan bir nechta muhim servislarni tasdiqladik, endi ulardan foydalanib domenni yanada chuqurroq enumeratsiya qilishimiz mumkin.
