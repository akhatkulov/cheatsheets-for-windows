## 🔐 **Active Directory (AD) nima va nima uchun muhim?**

Active Directory (AD) — bu Microsoft tomonidan ishlab chiqilgan xizmatlar to‘plami bo‘lib, Windows tarmog‘ini markazlashtirilgan tarzda boshqarishga imkon beradi. Dunyo bo‘yicha Global Fortune 1000 kompaniyalarining 90% dan ortig‘i undan foydalanadi. Windows tizimi bo‘lgan har qanday tashkilotda deyarli AD bo'lishi aniq. Shuning uchun u "qirollik kalitlari" sifatida qaraladi — ya'ni, tarmoqdagi barcha kirishlar, rollar va resurslarni boshqaradi. Bu esa uni **hujumchilar uchun asosiy nishon**ga aylantiradi.

**Maslahat:** AD’ni chuqurroq o‘rganish uchun, avval `AD basics` xonasini (TryHackMe'dagi) tugatib chiqing.

---

## 🎯 **Active Directory tizimiga kirib borish (breach)**

AD zaifliklaridan foydalanib tizimni ekspluatatsiya qilishdan oldin, **dastlabki kirish** kerak bo‘ladi — ya’ni haqiqiy foydalanuvchi ma'lumotlarini qo‘lga kiritish.

### 💡 Muhim narsa:

* Bizga yuqori huquqli hisob kerak emas.
* **Oddiy foydalanuvchi (low-privileged)** hisobining login ma'lumotlari ham yetarli.
* Maqsad: AD tizimiga ulanish va keyinchalik ichki tarmoqni tahlil qilish.

---

## 📚 **Ushbu darsda ko‘rib chiqiladigan hujum usullari:**

1. **NTLM authentication services** — NTLM protokoli orqali kirish
2. **LDAP bind credentials** — LDAP orqali ulanishda ishlatiladigan login ma'lumotlari
3. **Authentication relay** — Avtentifikatsiyani boshqa joyga yo‘naltirish
4. **Microsoft Deployment Toolkit** — MDK’dagi zaifliklar orqali
5. **Configuration files** — konfiguratsiya fayllaridan parollarni topish

Bu usullar orqali kompaniya tarmog‘iga tashqi yoki ichki yo‘l bilan kirish mumkin:

* Tashqi hujum (internetga ochiq tizimlar orqali)
* Ichki hujum (masalan, kompaniya tarmog‘iga ulangan qurilma orqali)

---

## 🌐 **Tarmoqqa ulanish (AttackBox va VPN)**

### 🔹 Agar siz TryHackMe’dagi AttackBox’dan foydalansangiz:

* Siz avtomatik tarzda tarmoqqa ulanishingiz kerak.
* Quyidagi buyruq orqali ulanishni tekshirishingiz mumkin:

```bash
ping thmdc.za.tryhackme.com
```

### ❗ Ammo DNS sozlamasini o‘zgartirish zarur:

```bash
sed -i '1s|^|nameserver $THMDCIP\n|' /etc/resolv-dnsmasq
```

Bu yerda `$THMDCIP` — sizga berilgan domen kontrollerining IP manzili.

Tekshiruv:

```bash
nslookup thmdc.za.tryhackme.com
```

Agar IP qaytsa — DNS ishlayapti.

> **Eslatma:** DNS har 3 soatda reset bo‘lishi mumkin — shuning uchun uni qayta sozlash kerak bo‘ladi.

---

## 📥 **VPN orqali o‘z qurilmangizdan ulanmoqchi bo‘lsangiz:**

1. `breachingad.ovpn` faylini yuklab oling.
2. Terminalda quyidagini ishga tushiring:

```bash
sudo openvpn breachingad.ovpn
```

Agar quyidagi xabar chiqsa:

```
Initialization Sequence Completed
```

Demak, siz tarmoqqa ulandingiz.

> **Eslatma:** Bu holda ham DNS sozlashingiz kerak.

---

## ⚙️ **Kali Linux’da DNS sozlash:**

1. GUI orqali:

   * Network Manager -> Advanced -> IPv4 settings -> DNS ni qo‘shing
   * THMDC IP manzilini birinchi, 1.1.1.1 yoki 8.8.8.8 ni ikkinchi qilib kiriting
2. So‘ng:

```bash
sudo systemctl restart NetworkManager
```

DNS ishlayotganini quyidagicha test qiling:

```bash
nslookup thmdc.za.tryhackme.com
```

---

## 🛠 **DNS bilan bog‘liq muammolarni tahlil qilish**

DNS Active Directory uchun **asosiy komponent** hisoblanadi, ayniqsa Kerberos tizimida. Chunki Kerberos faqat **hostname orqali** ishlaydi, **IP orqali emas**.

### 🔍 DNS muammo bo‘lsa, quyidagilarni bajaring:

1. **ping <THMDC IP>** — tarmoq ishlayotganini tekshirish
2. **nslookup za.tryhackme.com <THMDC IP>** — DNS server ishlayotganini aniqlash
3. **nslookup tryhackme.com** — bu umumiy internet DNS’ni tekshiradi

Agar bu ishlamasa:

* DNS manzilingiz noto‘g‘ri bo‘lishi mumkin
* `/etc/resolv.conf` faylida `nameserver` noto‘g‘ri joylashgan bo‘lishi mumkin
* Qayta sozlang

---

## 📌 **Yakuniy eslatma**

Bu `Breaching AD` xonasi o‘rtacha darajada (`medium`) baholangan. AD murakkab tizim bo‘lib, sizdan mustaqil fikrlash, tahlil qilish va yechim izlashni talab qiladi.

DNS bilan bog‘liq muammolar bu kabi muhitda juda ko‘p uchraydi — shuning uchun ularni tahlil qilishni o‘rganish **sizning kiberxavfsizlikdagi muvaffaqiyatingiz uchun juda muhim**.
