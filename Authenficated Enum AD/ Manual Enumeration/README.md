
Ushbu topshiriqda **CMD va PowerShell’ning o‘ziga xos (native) buyruqlari** yordamida **qo‘lda enumeratsiya** qilishga e’tibor qaratiladi. Bunga token imtiyozlarini (token privileges) tekshirish, domen va lokal foydalanuvchilarni (jumladan service account’lar) aniqlash, tizimda kirib turgan sessiyalarni tahlil qilish hamda environment variable’lar va Windows Registry’dan foydali ma’lumotlarni yig‘ish kiradi.

Demak, siz Windows mashinada shell qo‘lga kiritdingiz va atrof-muhitni o‘rganmoqchisiz. Grafik interfeysga o‘rganganlar uchun komandalar satri avvaliga qo‘rqinchli tuyulishi mumkin; ammo maqsadingiz uchun kerakli asosiy buyruqlarni o‘rganganingizdan so‘ng, uning moslashuvchanligi va kuchini yoqtirib qolasiz.

Tasavvur qiling: siz tunda notanish ofis binosida qoldingiz. Nima qilardingiz? Penetratsion testchi (albatta yozma ruxsat bilan) fikrlashi bilan “kashfiyot” sayohatini boshlaysiz. Masalan, qaysi ofisga tushganingizni bilish uchun nomini tekshirasiz, boshqa xonalarda eshikdagi lavhalarga qaraysiz, qaysi eshiklar ochiq ekanini tekshirasiz. **Manual enumeration** — kiber olamdagi shunga o‘xshash jarayon: siz kimsiz, yana kimlar bor, va qaysi “eshiklar” (imtiyozlar/huquqlar) sizga ochiq ekanini aniqlaysiz.

Bir ma’noda siz detektiv bo‘lasiz: qayerdaligingizni va nimalarga qodir ekaningizni topishingiz kerak. Bu topshiriqda imkon qadar “shovqinsiz” tarzda ma’lumot yig‘ish uchun **CMD va PowerShell** kabi ichki vositalardan foydalanasiz. MS Windows’ning o‘z ichki buyruqlaridan foydalangani uchun bu yondashuv **living off the land (LOTL)** deb ataladi. Hujumchilar kabi, pen-testchilar ham ko‘pincha `net` kabi native buyruqlar bilan foydalanuvchilarni, guruhlarni va konfiguratsiyalarni aniqlab, ko‘pincha ogohlantirishlarni qo‘zg‘atmasdan ish yuritadi. Ushbu topshiriqda tizim va domenni “xaritaga tushirish” uchun kerakli buyruqlar ko‘rib chiqiladi.

---

## Men kimman?

Har qanday hujumchi (yoki faylasuf) beradigan birinchi savol: **“Men kimman?”** Kompyuterda bunga `whoami` javob beradi.

Davom etish uchun AttackBox terminalida **10.211.12.20** IP manzildagi Workstation’ga SSH orqali ulang. Quyidagi credential’lardan foydalanamiz:

* **Username:** asrepuser1
* **Password:** qwerty123!

SSH ulanish namunasi:

```
ssh asrepuser1@10.211.12.20
```

Agar terminalda `tryhackme\asrepuser1` ko‘rsangiz — bu domen va username; `WRK` esa host nomi (hostname). Endi quyidagini ishga tushiring:

```
whoami
```

Bu buyruq joriy foydalanuvchining username’ini va (bo‘lsa) domenini ko‘rsatadi. Ya’ni, agar `DomainName\DomainUser` ko‘rinsa — bu domen akkaunti. Agar `ComputerName\LocalUser` bo‘lsa — bu lokal akkaunt.

Akkauntning guruhlari va imtiyozlarini bilish uchun `whoami /allýall` ishlatiladi:

```
whoami /all
```

Bu buyruq juda ko‘p ma’lumot beradi: **SID**, guruh a’zoliklari, va **privileges** (huquq/imtiyozlar). Guruhlarni ko‘rib, siz qiziqarli guruhlarda bormisiz (masalan, local Administrators, Domain Users, Backup Operators) aniqlaysiz. Ba’zan Domain Admins bo‘lib qolish ham mumkin — bu esa tizim juda noto‘g‘ri sozlanganini bildiradi.

Masalan, hamma foydalanuvchida bo‘ladigan `SeChangeNotifyPrivilege` kabi oddiy imtiyozlar, yoki service account / yuqori huquqli kontekstda bo‘lishi mumkin bo‘lgan `SeImpersonatePrivilege`, `SeAssignPrimaryTokenPrivilege` kabi muhim imtiyozlar chiqishi mumkin.

Nega bu muhim? Ba’zi imtiyozlar “saltanat kaliti” bo‘lishi mumkin. Masalan, `SeImpersonatePrivilege` autentifikatsiyadan keyin boshqa foydalanuvchini impersonatsiya qilishga imkon beradi. Bu juda katta narsa: token impersonation eksploitlari orqali SYSTEM shell olish ehtimoli bor. “Potato” turidagi hujumlar aynan shunday imtiyozlardan foydalanadi.

---

## Privileges (muhim imtiyozlar)

Keyingi qadamni rejalashtirishda juda muhim bo‘lishi mumkin bo‘lgan ayrim imtiyozlar:

* **SeImpersonatePrivilege:** “potato” hujumlari kabi token impersonation’ni suiste’mol qilishga olib kelishi mumkin.
* **SeAssignPrimaryTokenPrivilege:** boshqa foydalanuvchi primary token’ini yangi process’ga biriktirish imkonini beradi; ko‘pincha SeImpersonate bilan bog‘liq ishlatiladi.
* **SeBackupPrivilege:** fayl ruxsatlariga qaramasdan har qanday faylni o‘qishga ruxsat beradi; SAM yoki SYSTEM hive kabi sezgir fayllarni olishga yordam berishi mumkin.
* **SeRestorePrivilege:** fayl/registry ruxsatlaridan qat’i nazar yozish imkonini beradi; kritik tizim fayllari yoki registry sozlamalarini ustidan yozish xavfi bor.
* **SeDebugPrivilege:** istalgan process’ga debugger ulash imkonini beradi; LSASS xotirasini dump qilib credential olish yoki privileged process’ga kod kiritish (injection) uchun ishlatilishi mumkin.

Xulosa: `whoami /all` sizning “quvvat darajangiz”ni ko‘rsatadi (guruhlar va imtiyozlar orqali). Topilmalaringizni qayd etib boring — bu sizning boshlang‘ich nuqtangiz.

---

## Tizim va domen haqida ma’lumot

Endi siz kimsiz — bilasiz. Endi esa turgan tizimingiz qanday?

Asosiy savollar:

* Bu mashina Active Directory domeniga ulanganmi?
* Hostname nima?
* Domen yoki workgroup nomi qanday?

Quyidagi buyruqlar yordam beradi:

* `hostname`
* `systeminfo` (ko‘pincha admin huquqi kerak bo‘ladi)
* `set`

`hostname` kompyuter nomini chiqaradi. Ko‘pincha nomdan server roli haqida ham taxmin qilish mumkin (masalan, `dc` — domain controller bo‘lishi ehtimoli).

`systeminfo` OS versiyasi, hotfix’lar, domen/workgroup kabi juda ko‘p ma’lumot beradi. Filtrlash misollari:

* OS haqida:
  `systeminfo | findstr /B "OS"`
* Domen haqida:
  `systeminfo | findstr /B "Domain"`

`set` esa environment variable’larni ko‘rsatadi: user home, user domain va boshqalar. E’tibor: kompyuter domen a’zosi bo‘lmasa `USERDOMAIN` odatda kompyuter nomiga teng bo‘ladi. PowerShell’da `set` o‘rniga `Get-ChildItem Env:` yoki `dir env:` ishlatishingiz mumkin.

---

## NET buyruqlari bilan user va group enumeratsiyasi (CMD)

Hozircha siz:

* hostda kimsiz,
* domen bormi,
* guruhlar va privileges’ni bilasiz.

Endi domenning o‘zini enumeratsiya qilamiz: user’lar va group’lar.

**NET** — tarmoq resurslarini ko‘rish/boshqarish uchun buyruqlar to‘plami bo‘lib, har bir Windows’da bor. Ko‘pincha alohida tool’lar o‘rnatmasdan user/group/computer ma’lumotlarini olishga yetadi. Advanced hujumchilar ham recon’da NET’dan foydalanadi, chunki bu admin faoliyatiga o‘xshab ko‘rinadi.

Barcha buyruqlar ro‘yxati:

```
net help
```

---

### Domen user’lari

Domen user’larini chiqarish:

```
net user /domain
```

`net user` — lokal user’larni, `net user /domain` esa domen controller’dan domen user’larini oladi. Domen katta bo‘lsa, ro‘yxat uzun chiqishi va vaqt olishi mumkin.

Muayyan user haqida batafsil:

```
net user <username> /domain
```

Bu buyruq Full Name, account holati, parol muddati, group a’zoliklari, last logon kabi ma’lumotlarni beradi.

---

### Domen group’lari

Domen group’larini chiqarish:

```
net group /domain
```

Qiziqarli group’lar:

* **Domain Admins / Administrators** — butun AD uchun “kalit”
* **Enterprise Admins** — multi-domain forest’da juda kuchli
* **Server Operators / Backup Operators** — imtiyozli built-in group’lar
* Nomida “Admin” bor group’lar (masalan “SQL Admins”) ham ko‘pincha qiziqarli.

Guruh a’zolarini ko‘rish:

```
net group "Domain Admins" /domain
```

Kompyuter akkauntlarini ko‘rish (machine account’lar odatda `$` bilan tugaydi):

```
net group "Domain Computers" /domain
```

---

### Lokal group’lar

Lokal group’lar ro‘yxati:

```
net localgroup
```

Muayyan lokal group a’zolari (masalan Administrators):

```
net localgroup administrators
```

---

## Logged-on user’lar va sessiyalar

Ofis analogiyasi bo‘yicha: **“Bu yerda yana kim bor?”**

Logged-on user’larni ko‘rish:

```
quser
```

Bu orqali kim console’da, kim RDP orqali kirganini bilish mumkin. Agar administrator kirib turgan bo‘lsa — bu katta topilma: xotirada token/credential bo‘lish ehtimoli yuqori. Shunga o‘xshash holatda hujumchi LSASS dump, token o‘g‘irlash yoki Mimikatz kabi yo‘nalishlarga qiziqadi (albatta pentest doirasida).

Yana foydali buyruqlar (ko‘pincha elevated talab qiladi):

* `tasklist` — running process’lar
* `tasklist /V` — verbose
* `net session` — SMB sessiyalar (admin talab qiladi)

---

## Service account’larni aniqlash

Hammasi ham “inson user” emas — ko‘plari service account. Ularning parollari ko‘pincha o‘zgarmaydi va expire bo‘lmaydi, chunki xizmat to‘xtab qolishi mumkin.

### WMIC orqali

Xizmatlar va ularni ishga tushirgan akkaunt:

```
wmic service get Name,StartName
```

`StartName` — service qaysi akkaunt ostida ishlayotganini ko‘rsatadi. Odatdagi qiymatlar:

* `LocalSystem`
* `NT AUTHORITY\LocalService`
* `NT AUTHORITY\NetworkService`
* `NT SERVICE\...` (virtual service account)

Ba’zan `DomainName\username` bo‘lsa — bu ayniqsa qiziqarli (qayta ishlatilishi yoki zaif parol bo‘lishi mumkin).

PowerShell ekvivalenti:

```
Get-WmiObject Win32_Service | select Name, StartName
```

---

### SC orqali

Barcha service’larni chiqarish (uzun ro‘yxat, admin kerak bo‘lishi mumkin):

```
sc query state= all
```

Filtrlash:

```
sc query state= all | find "DHCP"
```

Service konfiguratsiyasini ko‘rish:

```
sc qc <ServiceName>
```

`SERVICE_START_NAME` — service qaysi akkaunt bilan ishlayotganini ko‘rsatadi.

---

## Environment va Registry’ni ko‘rish

`set` buyruqida environment variable’lar ko‘p narsa aytadi. Masalan `JAVA_HOME` kabi o‘zgaruvchilar o‘rnatilgan dasturlar va dev tool’lar haqida ishora beradi.

Windows Registry esa juda katta, hammasini ko‘rib chiqish amaliy emas. Lekin ayrim joylar juda foydali.

---

### Auto-logon (saqlangan avtomatik login) credential’lar

Ba’zi noto‘g‘ri sozlangan tizimlarda auto-logon paroli saqlangan bo‘lishi mumkin. Joylashuvi:

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

Masalan DefaultUsername’ni tekshirish:

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUsername
```

Agar saqlangan bo‘lsa, **DefaultPassword** plaintext bo‘lishi mumkin. `AutoAdminLogon` 1 bo‘lsa auto-logon yoqilgan bo‘ladi.

Yana bir joy: `HKLM\Security\Cache` — admin talab qiladi, credential’lar xesh ko‘rinishida bo‘ladi va cracking talab qilishi mumkin.

---

### O‘rnatilgan ilovalar

O‘rnatilgan dasturlar ro‘yxatini olish:

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
```

Bu ayniqsa default credential’lari bor eski/zaif dasturlarni topishda foydali bo‘lishi mumkin.

---

### Registry’da qidirish

Masalan, registry’dan “password” so‘zini qidirish:

```
reg query HKLM /f "password" /t REG_SZ /s
```

---

## Scheduled Tasks

Scheduled task’larni ko‘rish:

```
schtasks /query
```

Eslatma: `schtasks` orqali task yaratish (`/create`) yoki ishga tushirish (`/run`) ham mumkin.

---

## Ulanish bo‘yicha eslatma

SSH client orqali **10.211.12.20** IP manzildagi Workstation’ga ulanishingiz kerak. Domen akkaunt ma’lumotlari:

* **Username:** asrepuser1
* **Password:** qwerty123!

---
