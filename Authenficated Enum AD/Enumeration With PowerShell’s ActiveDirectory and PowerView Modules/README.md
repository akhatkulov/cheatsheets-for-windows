

### 3-vazifa: Windows PowerShell yordamida Enumeration

3-vazifada biz asosan **Command Prompt** (buyruq satri) dan foydalandik. Buyruq satri juda foydali bo‘lsa-da, **Windows PowerShell** dan foydalansak, yanada samaraliroq ishlashimiz mumkin. Ushbu vazifa ikki qismdan iborat:

* Birinchi qismda **rasmiy ActiveDirectory moduli** bilan tanishamiz
* Ikkinchi qismda esa **PowerSploit framework** tarkibidagi **PowerView moduli** dan foydalanamiz

---

## ActiveDirectory moduli bilan Enumeration

**ActiveDirectory moduli** domen kontrollerlarida (Domain Controller) mavjud bo‘ladi. Boshqa ishchi stansiyalar va serverlarda esa uni ishlatish uchun **Remote Server Administration Tools (RSAT)** ni yuklab olish kerak bo‘ladi. Muqobil ravishda, ayrim repozitoriylarda ActiveDirectory modulini to‘liq RSAT o‘rnatmasdan ham yuklab olish mumkin.

### Ulanish

Amaliyotni davom ettirish uchun **AttackBox** terminalidan foydalanib, quyidagi IP manzildagi **Workstation** ga SSH orqali ulanamiz:

* **IP**: `10.211.12.20`
* **Username**: `asrepuser1`
* **Password**: `qwerty123!`

Ulanish namunasi:

```text
ssh asrepuser1@10.211.12.20
```

Muvaffaqiyatli ulanganingizdan so‘ng, Windows Command Prompt ochiladi.

---

### PowerShell’ni ishga tushirish

Endi PowerShell’ni ishga tushiramiz:

```text
powershell
```

PowerShell prompti `PS` bilan boshlanishini ko‘rasiz — bu Command Prompt’dan PowerShell’ga o‘tganingizni bildiradi.

---

### ActiveDirectory modulini tekshirish va yuklash

Modul mavjudligini tekshirish:

```powershell
Get-Module -ListAvailable ActiveDirectory
```

Agar modul mavjud bo‘lsa, uni quyidagicha yuklaymiz:

```powershell
Import-Module ActiveDirectory
```

---

## Foydalanuvchilarni aniqlash (User Enumeration)

Barcha foydalanuvchilarni ko‘rish:

```powershell
Get-ADUser -Filter *
```

Bu buyruq domen ichidagi barcha foydalanuvchi akkauntlarini chiqaradi.

---

### Muayyan foydalanuvchi haqida ma’lumot olish

Bitta foydalanuvchi haqida ma’lumot:

```powershell
Get-ADUser -Identity <username>
```

Agar foydalanuvchining **barcha xususiyatlarini** ko‘rmoqchi bo‘lsangiz:

```powershell
Get-ADUser -Identity <username> -Properties *
```

---

### Muhim xususiyatlarni tanlab ko‘rish

Ko‘pincha quyidagi maydonlar juda foydali bo‘ladi:

* **LastLogonDate** – oxirgi kirish vaqti
* **MemberOf** – qaysi guruhlarga a’zo
* **Description** – izoh (qimmatli ma’lumot bo‘lishi mumkin)
* **Title** – lavozimi
* **PwdLastSet** – parol oxirgi marta qachon o‘zgartirilgan

Misol:

```powershell
Get-ADUser -Identity Administrator -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
```

---

### Filtrlash

Nomida `admin` so‘zi bor foydalanuvchilarni topish:

```powershell
Get-ADUser -Filter "Name -like '*admin*'"
```

---

## Guruhlarni aniqlash (Group Enumeration)

Barcha guruhlarni ko‘rish:

```powershell
Get-ADGroup -Filter *
```

Faqat guruh nomlarini chiqarish:

```powershell
Get-ADGroup -Filter * | Select Name
```

---

### Guruh a’zolarini ko‘rish

Muayyan guruh a’zolari:

```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

Bu buyruq **Domain Admins** guruhiga kiruvchi barcha akkauntlarni ko‘rsatadi.

---

## Kompyuterlarni aniqlash (Computer Enumeration)

Domen ichidagi barcha kompyuterlar:

```powershell
Get-ADComputer -Filter *
```

Faqat nomi va operatsion tizimini ko‘rish:

```powershell
Get-ADComputer -Filter * | Select Name, OperatingSystem
```

---

## Boshqa foydali buyruqlar

### Domen parol siyosati

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Bu buyruq quyidagi ma’lumotlarni ko‘rsatadi:

* Parol uzunligi
* Murakkablik talabi
* Bloklash vaqti
* Parol amal qilish muddati

---

## PowerView bilan Enumeration

### PowerView nima?

**PowerView** — bu **PowerSploit framework** tarkibidagi PowerShell asosidagi vosita bo‘lib, domenni chuqur tahlil qilish (enumeration) uchun ishlatiladi. U `net user` va `net group` kabi buyruqlarning ancha rivojlangan shakli hisoblanadi.

PowerSploit quyidagi manzilda mavjud:

```text
C:\Users\asrepuser1\Downloads\PowerSploit-master
```

PowerView skripti esa:

```text
C:\Users\asrepuser1\Downloads\PowerSploit-master\Recon\PowerView.ps1
```

---

### PowerView’ni yuklash

Recon papkasiga o‘ting va modulni yuklang:

```powershell
Import-Module .\PowerView.ps1
```

Agar xatolik chiqmasa — modul muvaffaqiyatli yuklangan.

---

## PowerView bilan foydalanuvchilarni aniqlash

Barcha domen foydalanuvchilari:

```powershell
Get-DomainUser
```

Nomida `admin` bo‘lgan foydalanuvchilar:

```powershell
Get-DomainUser *admin*
```

---

## Guruhlarni aniqlash (PowerView)

Barcha guruhlar:

```powershell
Get-DomainGroup
```

Nomida `admin` bo‘lgan guruhlar:

```powershell
Get-DomainGroup "*admin*"
```

---

## Kompyuterlarni aniqlash (PowerView)

Domen ichidagi barcha kompyuterlar:

```powershell
Get-DomainComputer
```

---

## Qo‘shimcha foydali PowerView buyruqlari

* **Administrator huquqiga ega foydalanuvchilar**:

  ```powershell
  Get-DomainUser -AdminCount
  ```

* **SPN (Service Principal Name) mavjud akkauntlar**
  (Kerberoasting hujumlari uchun muhim):

  ```powershell
  Get-DomainUser -SPN
  ```

---

### Xulosa

PowerView va ActiveDirectory PowerShell moduli:

* Juda kuchli va moslashuvchan
* Keng filtrlash imkoniyatlariga ega
* `net user` va `net group` buyruqlariga qaraganda ancha qulay

Qo‘shimcha buyruqlar va imkoniyatlar uchun rasmiy hujjatlarni o‘rganish tavsiya etiladi.

