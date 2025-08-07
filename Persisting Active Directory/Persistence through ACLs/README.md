
### Ba’zida oddiy AD guruhlarga doimiylikni o‘rnatish yetarli bo‘lmaydi. Agar biz barcha **himoyalangan guruhlarga** bir vaqtning o‘zida doimiylikni o‘rnatmoqchi bo‘lsak-chi?

## **AD Guruh Shablonlari Orqali Doimiylik**

Biz boshqaruvidagi akkauntni topilgan har bir imtiyozli guruhga qo‘shishimiz mumkin, lekin **Blue Team** (himoya jamoasi) hali ham a’zolikni olib tashlab, tozalash ishlarini bajarishi mumkin. Biroq, agar biz doimiylikni **shablonlar orqali** o‘rnatsak, bu holatda Blue Team a’zolikni olib tashlasa ham, **shablon yangilangach, biz yana guruhga qaytadan qo‘shilamiz**.

Shunday shablonlardan biri bu **AdminSDHolder konteyneri**. Bu konteyner har bir AD domenida mavjud bo‘lib, uning **Access Control List (ACL)** ro‘yxati barcha himoyalangan guruhlar uchun shablon sifatida ishlatiladi. Bunday himoyalangan guruhlarga quyidagilar kiradi:

* Domain Admins
* Administrators
* Enterprise Admins
* Schema Admins

To‘liq ro‘yxatni esa [bu yerda](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/admin-sdholder) topish mumkin.

AD ichida **SDProp** nomli xizmat mavjud bo‘lib, u **AdminSDHolder konteyneridagi ACL** ni har 60 daqiqada himoyalangan guruhlarga nusxalab beradi. Agar biz bu konteynerga o‘zimizga **to‘liq ruxsat beruvchi ACE (Access Control Entry)** qo‘shsak, bu doimiylikni ta’minlaydi. Blue Team bunga e’tibor bermasa, ular ruxsatni olib tashlaganlaridan keyin ham **60 daqiqadan so‘ng biz yana shu ruxsatga ega bo‘lamiz**. Bu esa ular uchun chalkashlik va bosh og‘rig‘iga sabab bo‘ladi. Bu jarayon odatiy AD xizmatlari orqali bo‘lib o‘tadi, shuning uchun hech qanday ogohlantirishlar ko‘rsatilmaydi.

---

## **AdminSDHolder orqali doimiylikni o‘rnatish**

Endi bu doimiylikni o‘rnatish uchun biz **Microsoft Management Console (MMC)** dan foydalanamiz.

1. **Low-privileged** foydalanuvchi sifatida **THMWRK1** ga RDP orqali ulanamiz.
2. Administrator huquqiga ega bo‘lish uchun `runas` buyrug‘idan foydalanamiz:

```bash
runas /netonly /user:thmchilddc.tryhackme.loc\Administrator cmd.exe
```

3. Ochiqlgan yangi oynada `mmc` deb yozamiz va kiramiz.
4. `File -> Add Snap-In -> Active Directory Users and Computers` menyusi orqali **Users and Groups** Snap-in ni qo‘shamiz.
5. `View -> Advanced Features` ni yoqamiz.
6. **Domain -> System** ichida **AdminSDHolder** ni topamiz.
7. Unga o‘ng tugma -> Properties -> **Security** bo‘limiga o‘tamiz.
8. Quyidagi qadamlar bilan low-privileged foydalanuvchini to‘liq ruxsat bilan qo‘shamiz:

   * "Add" tugmasini bosing.
   * Foydalanuvchingiz nomini kiriting, "Check Names" bosing.
   * "OK" tugmasini bosing.
   * "Full Control" ni belgilab, "Allow" tugmasini bosing.
   * "Apply" va so‘ng "OK" ni bosing.

---

## **SDProp xizmatini qo‘lda ishga tushurish**

Biz 60 daqiqa kutishni xohlamasligimiz mumkin. Buning o‘rniga SDProp xizmatini PowerShell orqali qo‘lda ishga tushirishimiz mumkin. Bu uchun `C:\Tools\` papkasida tayyor skript mavjud:

```powershell
PS C:\Tools> Import-Module .\Invoke-ADSDPropagation.ps1
PS C:\Tools> Invoke-ADSDPropagation
```

Keyin, bir necha daqiqa kuting va `Domain Admins` guruhining **Security** sozlamalarini tekshiring — sizning foydalanuvchingizda **to‘liq nazorat** borligini ko‘rasiz. Hatto sizni Security ruxsatlaridan olib tashlasalar ham, skriptni yana ishga tushirsangiz, u avtomatik qayta tiklanadi.

Shuni unutmang: ruxsatga ega bo‘lish **sizni avtomatik guruh a’zosi qilmaydi**, lekin bu ruxsatlar yordamida **o‘zingizni guruhga qo‘sha olasiz**.

---

## **Blue Team uchun og‘ir kunlar boshlanmoqda**

Agar siz bu texnikani **ilgari o‘rganilgan guruh ichida guruh qo‘shish (nested groups)** texnikasi bilan birlashtirsangiz, Blue Team sizni tozalaganidan keyin yana 60 daqiqa o‘tib hammasini qayta boshlashingiz mumkin. Agar ular **AdminSDHolder orqali doimiylik o‘rnatilganini** bilmasa, bu juda katta muammo bo‘ladi.

Hatto shunday qilish ham mumkin: **AdminSDHolder** guruhida `Domain Users` guruhiga **Full Control** ruxsati berilsa, har qanday oddiy foydalanuvchi barcha himoyalangan guruhlarga to‘liq nazoratga ega bo‘ladi.

Agar siz bunga **DC Sync** imkoniyatini ham qo‘shsangiz, Blue Team sizni butunlay tizimdan chiqarish uchun **domen ichidagi barcha parollarni yangilashi kerak bo‘ladi**.
