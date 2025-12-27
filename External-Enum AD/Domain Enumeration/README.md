Endi maqsad tarmog‘i haqida yaxshiroq tasavvurga ega bo‘lganimizdan so‘ng, haqiqiy akkauntlarni va potensial nishonlarni aniqlash uchun **foydalanuvchilarni enumeratsiya qilish** vaqti keldi. Username’larni turli xil **autentifikatsiyasiz** usullar bilan yig‘ish mumkin — har bir usul turli noto‘g‘ri sozlamalar (misconfigurations)ga tayanadi.

Ushbu topshiriqda biz muvaffaqiyatli user enumeration uchun kerak bo‘ladigan vositalar va texnikalarni ko‘rib chiqamiz.

---

## LDAP Enumeration (Anonymous Bind)

**Lightweight Directory Access Protocol (LDAP)** — Microsoft Active Directory kabi directory servislariga kirish va ularni boshqarish uchun keng qo‘llaniladigan protokol. LDAP tarmoq ichidagi resurslarni (foydalanuvchilar, guruhlar, qurilmalar va tashkilotga oid ma’lumotlar) topish va tartibga solishga yordam beradi, chunki u markaziy directory’ni taqdim etadi va ilovalar hamda foydalanuvchilar unga so‘rov yubora oladi.

Ba’zi LDAP serverlar anonim foydalanuvchilarga faqat o‘qish (read-only) rejimida so‘rov yuborishga ruxsat beradi. Bu esa user account’lar va boshqa directory ma’lumotlarini ochib qo‘yishi mumkin.

Anonim LDAP bind yoqilgan-yo‘qligini `ldapsearch` bilan tekshirish mumkin:

```
ldapsearch -x -H ldap://10.211.11.10 -s base
```

* `-x`: Simple authentication (bizning holatda — anonim autentifikatsiya)
* `-H`: LDAP server manzilini ko‘rsatadi
* `-s`: so‘rovni faqat base obyekt bilan cheklaydi (subtree/child’larni qidirmaydi)

Agar u yoqilgan bo‘lsa, quyidagiga o‘xshash ko‘p ma’lumot chiqishini ko‘ramiz:

### Terminal

```
user@tryhackme$ ldapsearch -x -H ldap://10.211.11.10 -s base
dn:
domainFunctionality: 6
forestFunctionality: 6
domainControllerFunctionality: 7
rootDomainNamingContext: DC=tryhackme,DC=loc
ldapServiceName: tryhackme.loc:dc$@TRYHACKME.LOC
isGlobalCatalogReady: TR
dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN
 =Sites,CN=Configuration,DC=tryhackme,DC=loc
dnsHostName: DC.tryhackme.loc
defaultNamingContext: DC=tryhackme,DC=loc
currentTime: 20250514173531.0Z
configurationNamingContext: CN=Configuration,DC=tryhackme,DC=loc
search result
search: 2
result: 0 Success
```

So‘ng foydalanuvchilar haqidagi ma’lumotlarni quyidagi buyruq bilan so‘rashimiz mumkin:

```
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```

---

## Enum4linux-ng

**enum4linux-ng** — Windows tizimlariga qarshi turli enumeratsiya texnikalarini avtomatlashtiruvchi vosita bo‘lib, user enumeration’ni ham bajaradi. U SMB va RPC protokollaridan foydalanib user ro‘yxatlari, guruh a’zoliklari, share ma’lumotlari va boshqa narsalarni yig‘adi.

Domain Controller’dan imkon qadar ko‘p ma’lumot olish uchun quyidagi buyruqni ishlatamiz:

```
enum4linux-ng -A 10.211.11.10 -oA results.txt
```

* `-A`: mavjud barcha enumeratsiya funksiyalarini bajaradi (users, groups, shares, password policy, RID cycling, OS info, NetBIOS info)
* `-oA`: natijani YAML va JSON fayllarga yozadi

---

## RPC Enumeration (Null Sessions)

**Microsoft Remote Procedure Call (MSRPC)** — bir kompyuterdagi dastur boshqa kompyuterdagi dasturdan servis so‘rashiga imkon beruvchi protokol bo‘lib, tarmoq tafsilotlarini bilish shart bo‘lmaydi. RPC servislariga SMB protokoli orqali ham kirish mumkin.

Agar SMB **null session**’larga (autentifikatsiyasiz) ruxsat berib qo‘yilgan bo‘lsa, autentifikatsiyasiz foydalanuvchi `IPC$` share’ga ulanib, tizim yoki domen bo‘yicha userlar, guruhlar, share’lar va boshqa sezgir ma’lumotlarni enumeratsiya qilishi mumkin.

Null session mavjudligini quyidagi buyruq bilan tekshiramiz:

```
rpcclient -U "" 10.211.11.10 -N
```

* `-U`: username ko‘rsatadi (biz anonim uchun bo‘sh string ishlatyapmiz)
* `-N`: RPC parol so‘ramasligini bildiradi

Agar muvaffaqiyatli bo‘lsa, userlarni quyidagicha enumeratsiya qilamiz: `enumdomusers`

### rpcclient

```
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[sshd] rid:[0x649]
user:[gerald.burgess] rid:[0x650]
user:[nigel.parsons] rid:[0x651]
user:[guy.smith] rid:[0x652]
user:[jeremy.booth] rid:[0x653]
user:[barbara.jones] rid:[0x654]
user:[marion.kay] rid:[0x655]
user:[kathryn.williams] rid:[0x656]
user:[danny.baker] rid:[0x657]
user:[gary.clarke] rid:[0x658]
user:[daniel.turner] rid:[0x659]
user:[debra.yates] rid:[0x65a]
user:[jeffrey.thompson] rid:[0x65b]
user:[martin.riley] rid:[0x65c]
user:[danielle.lee] rid:[0x65d]
user:[douglas.roberts] rid:[0x65e]
user:[dawn.bolton] rid:[0x65f]
user:[danielle.ali] rid:[0x660]
user:[michelle.palmer] rid:[0x661]
user:[katie.thomas] rid:[0x662]
user:[jennifer.harding] rid:[0x663]
user:[strategos] rid:[0x664]
user:[empanadal0v3r] rid:[0x665]
user:[drgonz0] rid:[0x666]
user:[strate905] rid:[0x667]
user:[krbtgtsvc] rid:[0x668]
user:[asrepuser1] rid:[0x669]
user:[rduke] rid:[0xa31]
```

`rpcclient` ichida `help` buyrug‘ini bajarib, mavjud buyruqlar ro‘yxatini ko‘rish mumkin. Yetarli ruxsatlar bo‘lsa, RPC orqali domenni ancha chuqur enumeratsiya qilish mumkin.

---

## RID Cycling

Active Directory’da **RID (Relative Identifier)** diapazonlari user va group obyektlariga noyob identifikatorlar berish uchun ishlatiladi. Bu RID’lar **SID (Security Identifier)** ning bir qismi bo‘lib, domen ichida har bir obyektni noyob tarzda belgilaydi. Ayrim RID’lar standart va “well-known”.

Masalan:

* **500** — Administrator akkaunti
* **501** — Guest akkaunti
* **512–514** — quyidagi guruhlar uchun: Domain Admins, Domain Users, Domain Guests
* Odatda user akkauntlari **RID 1000** dan boshlab ketadi

RID diapazonini enum4linux-ng bilan aniqlash mumkin, yoki ma’lum diapazondan (masalan 1000–1200) boshlab tekshirib, natija chiqsa kengaytirish mumkin.

Agar `enumdomusers` cheklangan bo‘lsa, har bir RID bo‘yicha userni alohida so‘rab ko‘rish uchun quyidagi bash buyruqni ishlatamiz:

### Terminal

```
user@tryhackme$ for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
	User Name   :	sshd
	User Name   :	gerald.burgess
	User Name   :	nigel.parsons
	User Name   :	guy.smith
	User Name   :	jeremy.booth
	User Name   :	barbara.jones 
```

Izoh:

* `for i in $(seq 500 2000)`: RID diapazonida yurib chiqadi
* `echo "queryuser $i"`: RID `$i` ga mos user haqida ma’lumot so‘raydi
* `2>/dev/null`: xato chiqishlarini jim qiladi
* `| grep -i "User Name"`: faqat “User Name” bo‘lgan satrlarni (katta-kichik harfni inobatga olmay) chiqaradi

Eslatma: bu buyruq bajarilishi **2–3 daqiqa** vaqt olishi mumkin.

---

## Kerbrute bilan Username Enumeration

**Kerberos** — Microsoft Windows domenlarida asosiy autentifikatsiya protokoli. NTLM challenge-response mexanizmiga tayanadigan bo‘lsa, Kerberos ishonchli uchinchi tomon (**KDC — Key Distribution Centre**) boshqaradigan ticket-based tizimdan foydalanadi. Bu yondashuv client va server o‘rtasida o‘zaro autentifikatsiyani ta’minlaydi hamda kuchliroq shifrlashdan foydalanadi, shu sababli odatda hujumlarga nisbatan chidamliroq.

**Kerbrute** — Kerberos pre-authentication’dan foydalanib, Active Directory’dagi haqiqiy userlarni bruteforce/enumeratsiya qilish uchun ommabop vosita.

enum4linux-ng yoki rpcclient ba’zi username’larni qaytarishi mumkin, ammo ular quyidagilar bo‘lishi ehtimoli bor:

* o‘chirilgan (disabled) akkauntlar
* domen akkaunti bo‘lmagan (non-domain) akkauntlar
* honeypot (soxta tuzoq) userlar
* yoki false positive’lar

Ularni kerbrute orqali tekshirib chiqish bizga qaysi userlar real va faol ekanini tasdiqlash imkonini beradi va password spray hujumlarini aniqroq nishonga olishga yordam beradi.

Oldingi vositalardan yig‘ilgan username’lar asosida user ro‘yxati tuzamiz:

### Misol Terminal

```
user@tryhackme$ cat users.txt
Administrator
Guest
krbtgt
sshd
gerald.burgess
nigel.parsons
...
asrepuser1
rduke
```

---

## Kerbrute o‘rnatish (Installation)

1. OS’ingiz uchun precompiled binary’ni yuklab oling:
   [https://github.com/ropnop/kerbrute/releases](https://github.com/ropnop/kerbrute/releases)

2. `kerbrute_linux_amd64` faylini `kerbrute` deb qayta nomlang.

3. `kerbrute`’ni executable qilish uchun:

```
chmod +x kerbrute
```

Eslatma: **Kerbrute AttackBox’da o‘rnatilmagan**, shuning uchun yuklab olish va sinab ko‘rish uchun internet kerak bo‘ladi.

---

## Kerbrute bilan username enumeratsiya qilish

Kerbrute Kerberos’ga qarshi username enumeration (bruteforce) bajaradi:

### Misol Terminal

```
user@tryhackme$ ./kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users.txt
```

(Quyidagi chiqish misolda ko‘rsatilgan)

Chiqishdan ko‘rinib turibdiki, ro‘yxatdagi 33 ta username’dan **28 tasi valid** ekan. Endi shu ma’lumot asosida user ro‘yxatini yangilashimiz mumkin.

Agar user enumeration uchun yagona variant kerbrute bo‘lsa (boshqa vositalar ishlamasa), u holda wordlist yuklab olib, undan foydalanuvchi akkauntlarini topish mumkin. Haqiqiy userlar ro‘yxatini topgach, ularni `users.txt` fayliga qo‘shamiz va keyingi **password spraying** hujumida ishlatamiz.
