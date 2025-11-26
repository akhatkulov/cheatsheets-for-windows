

## ❖ PowerShell

PowerShell — bu Command Prompt’ning takomillashtirilgan versiyasi. Microsoft uni birinchi marta **2006-yilda** chiqargan. PowerShell CMDdagi standart funksiyalarni o‘z ichiga oladi, lekin qo‘shimcha ravishda **cmdlet** (command-let) deb ataladigan maxsus .NET sinflaridan ham foydalanadi. Bu cmdletlar aniq funksiyalarni bajarish uchun yaratilgan. Xohlasa, biz ham o‘z cmdletlarimizni yozishimiz mumkin — masalan, PowerView yaratuvchilari shunday qilgan. Ammo ichki (built-in) cmdletlarning o‘zi bilan juda ko‘p masofani bosib o‘tish mumkin.

Biz Task 3’da **AD-RSAT** paketini o‘rnatganimiz sababli, unga tegishli PowerShell cmdletlar avtomatik tarzda o‘rnatildi. Ular **50+ ta**. Biz ulardan ba’zilarini ko‘rib chiqamiz, lekin to‘liq ro‘yxat uchun havolaga qarang.

SSH terminalidan foydalanayotgan bo‘lsak, uni quyidagi buyruq bilan PowerShell terminaliga aylantiramiz:

```
powershell
```

---

## 👤 Foydalanuvchilar (Users)

Active Directory foydalanuvchilarini enum qilish uchun `Get-ADUser` cmdletidan foydalanamiz:

```
Get-ADUser -Identity gordon.stevens -Server za.tryhackme.com -Properties *
```

Bu yerda:

* **-Identity** — qaysi akkaunt haqida ma’lumot olayapmiz
* **-Properties** — qaysi xususiyatlarni ko‘rsatish, `*` — hammasi
* **-Server** — biz domen-join bo‘lmaganimiz uchun domen controller manzili

Ko‘pchilik cmdletlarda **-Filter** parametri ham bor, bu batafsilroq qidiruv imkonini beradi. Masalan, ismida *stevens* so‘zi bor foydalanuvchilarni topish:

```
Get-ADUser -Filter 'Name -like "*stevens"' -Server za.tryhackme.com | Format-Table Name,SamAccountName -A
```

`Format-Table` natijani tartibli jadval ko‘rinishida chiqaradi.

---

## 👥 Guruhlar (Groups)

AD guruhlarini ko‘rish:

```
Get-ADGroup -Identity Administrators -Server za.tryhackme.com
```

Guruh a’zolarini olish:

```
Get-ADGroupMember -Identity Administrators -Server za.tryhackme.com
```

Natija:

* Userlar
* Boshqa guruhlar
* SamAccountName
* SID
* DistinguishedName

va hokazo.

---

## 🧱 AD obyektlari (AD Objects)

ADdagi har qanday obyektni qidirish uchun `Get-ADObject` ishlatiladi. Masalan, muayyan sanadan keyin o‘zgartirilgan obyektlarni topish:

```
$ChangeDate = New-Object DateTime(2022, 02, 28, 12, 00, 00)
Get-ADObject -Filter 'whenChanged -gt $ChangeDate' -includeDeletedObjects -Server za.tryhackme.com
```

Yoki password sprayingda account lock bo‘lishining oldini olish uchun `badPwdCount > 0` bo‘lganlarni aniqlash:

```
Get-ADObject -Filter 'badPwdCount -gt 0' -Server za.tryhackme.com
```

Bu faqat kimdir parolni noto‘g‘ri kiritgan bo‘lsa natija qaytaradi.

---

## 🌍 Domenlar (Domains)

Domen haqidagi qo‘shimcha ma’lumotlarni olish:

```
Get-ADDomain -Server za.tryhackme.com
```

Bu orqali:

* DNS nomi
* Default Users/Computers konteyneri
* Domain Controllers joylashuvi
* DistinguishedName
* Child domains

kabi ma’lumotlar olinadi.

---

## 🔧 AD obyektlarini o‘zgartirish

AD-RSAT cmdletlari orqali yangi obyekt yaratish yoki mavjudini o‘zgartirish mumkin. Ammo hozirgi tarmoqdagi vazifamiz **enum qilish** — ekspluatatsiya keyingi bo‘limda ko‘riladi.

Misol — parolni majburan o‘zgartirish:

```
Set-ADAccountPassword -Identity gordon.stevens -Server za.tryhackme.com -OldPassword (ConvertTo-SecureString -AsPlaintext "old" -force) -NewPassword (ConvertTo-SecureString -AsPlainText "new" -Force)
```

`Identity` va parollarni o‘zingizga berilgan ma’lumotga moslang.

---

## ✅ Afzalliklar

* PowerShell cmdletlari — CMDga qaraganda **juda ko‘p ma’lumot** beradi
* Domen-join bo‘lmagan mashinadan ham ishlatish mumkin (`-Server` orqali)
* Maxsus cmdletlar yaratish imkoniyati bor
* Parollarni o‘zgartirish, guruhlarga qo‘shish, obyekt yaratish — to‘g‘ridan-to‘g‘ri bajarish mumkin

---

## ⚠️ Kamchiliklar

* PowerShell odatda Blue Team tomonidan **ko‘proq kuzatiladi**
* AD-RSAT paketini o‘rnatish talab qilinadi — ayrim joylarda aniqlanishi mumkin
* Ba’zi enumeration skriptlari EDR tomonidan flag qilinadi

