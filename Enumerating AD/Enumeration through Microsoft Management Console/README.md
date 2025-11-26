

## **Siz hozirga qadar Active Directory Basics xonasini tugatgan bo‘lishingiz kerak**

U yerda turli AD obyektlari bilan dastlab tanishgansiz. Shu vazifada siz ularga nima ekanini allaqachon tushunasiz deb hisoblanadi.

Vazifani bajarish uchun **Task 1 dagi taqdim etilgan credentiallar bilan RDP orqali THMJMP1 ga uling**.

---

## **Microsoft Management Console**

Bu vazifada biz birinchi enumeratsiya usulini o‘rganamiz — bu so‘nggi vazifagacha yagona GUI’dan foydalaniladigan usuldir. Biz **Microsoft Management Console (MMC)** va Remote Server Administration Tools (RSAT) AD Snap-Ins’dan foydalanamiz.

Agar siz THMJMP1 Windows VM’dan foydalanayotgan bo‘lsangiz — u yerda allaqachon o‘rnatilgan.
Agar o‘zingizning Windows kompyuteringizdan foydalanayotgan bo‘lsangiz, quyidagilarni bajaring:

1. Start tugmasini bosing
2. “Apps & Features” ni qidiring va oching
3. “Manage Optional Features” ga kiring
4. “Add a feature” ni bosing
5. “RSAT” deb qidiring
6. **RSAT: Active Directory Domain Services and Lightweight Directory Tools** ni tanlab Install qiling

---

## **MMC-ni ishga tushirish**

Start → Run → `mmc` deb yozasiz.

Lekin agar MMC’ni oddiy ishga tushirsangiz, u ishlamaydi — chunki kompyuter domen-ga ulanmagan va lokal foydalanuvchi credentiallari bilan domen autentifikatsiya qilib bo‘lmaydi.

Bu joyda **oldingi vazifadagi Runas oynasi kerak bo‘ladi** — MMC’ni aynan o‘sha Runas CMD ichidan ishga tushirish kerak. Shunda MMC ichidagi barcha tarmoq bog‘lanishlar **inject qilingan AD credentiallari** bilan autentifikatsiya qilinadi.

---

## **AD RSAT Snap-In qo‘shish**

1. MMC ichida **File → Add/Remove Snap-in** ni bosing
2. Uchala Active Directory Snap-in’ni tanlab **Add** qiling
3. Xatolik yoki ogohlantirishlar chiqsa — davom eting
4. **Active Directory Domains and Trusts** → Right-click → Change Forest

   * `za.tryhackme.com` ni root domain sifatida kiriting → OK
5. **Active Directory Sites and Services** → Right-click → Change Forest

   * `za.tryhackme.com` ni kiriting → OK
6. **Active Directory Users and Computers** → Right-click → Change Domain

   * `za.tryhackme.com` ni kiriting → OK
7. Chap panelda **Active Directory Users and Computers** → Right-click → View → **Advanced Features** ni yoqing

Agar hammasi to‘g‘ri bajarilgan bo‘lsa, MMC endi maqsad domeniga ulangan va autentifikatsiya qilingan bo‘ladi.

---

## **Users va Computers bo‘yicha Enumeratsiya**

Active Directory strukturasini o‘rganamiz. Bu vazifada **AD Users and Computers**ga e’tibor qaratamiz.

Snap-in’ni oching → `za` domenini kengaytiring — boshlang‘ich **Organizational Unit (OU)** strukturasini ko‘rasiz.

### People direktori

Bu yerda foydalanuvchilar bo‘limlarga ajratilgan OUlarda guruhlangan.
Har bir OU’ni bosganda shu bo‘limga tegishli foydalanuvchilar chiqadi.

Har bir foydalanuvchini ikki marta bosganda:

✅ atributlari
✅ xususiyatlari
✅ qaysi guruh a’zosi ekanligi

ko‘rinadi.

---

## **Mashinalarni topish**

Servers yoki Workstations bo‘limiga kirilsa — domen-ga ulangan kompyuterlar ro‘yxati chiqadi.

Agar sizda kerakli ruxsatlar bo‘lsa, MMC orqali quyidagilarni ham qilishingiz mumkin:

* foydalanuvchi parolini o‘zgartirish
* akkauntni guruhlarga qo‘shish
* obyekt xususiyatlarini yangilash

Shuning uchun MMC bilan tajriba qilib ko‘ring, AD tuzilishini yaxshiroq tushunasiz.
Obyektlarni qidirish funksiyasidan foydalaning — juda qulay.

---

## **Afzalliklari**

✅ GUI AD muhitini umumiy manzara sifatida ko‘rish imkonini beradi
✅ AD obyektlarini tez qidirish mumkin
✅ Har bir obyektning yangilanishlarini bevosita ko‘rish oson
✅ Yetarli ruxsat bo‘lsa, to‘g‘ridan-to‘g‘ri AD obyektlarini tahrir qilish mumkin

---

## **Kamchiliklari**

❌ GUI faqat RDP orqali mavjud bo‘lgan mashinada ishlaydi
❌ Keng ko‘lamli AD atributlarini to‘plash yoki avtomatlashtirilgan enumeratsiya qiyin
❌ CLI yoki skriptlarga nisbatan sekinroq

