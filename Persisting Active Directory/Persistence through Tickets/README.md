## 🎟 Kerberos Tizimi: Tushuncha

**Kerberos** – bu Active Directory (AD) tarmoqlarida ishlatiladigan autentifikatsiya protokoli. Foydalanuvchi xizmatlarga kirishda, parol yubormasdan, tokenlar orqali isbotlaydi.

### 🛠 Oddiy Kerberos Ishlash Jarayoni:

1. **AS-REQ** – Foydalanuvchi DC (Domain Controller) ga so‘rov yuboradi, bu so‘rov NTLM hash bilan shifrlangan timestampni o‘z ichiga oladi.
2. **AS-REP** – DC foydalanuvchiga **TGT (Ticket Granting Ticket)** yuboradi. Bu TGT `KRBTGT` akkaunti hash’i bilan imzolangan bo‘ladi.
3. **TGS-REQ** – Foydalanuvchi bu TGT’ni olib, kerakli xizmatga kirish uchun DC’dan **TGS (Ticket Granting Service)** talab qiladi.
4. **TGS-REP** – DC xizmat uchun TGS yaratadi va yuboradi.
5. **Foydalanuvchi xizmatga TGS bilan kiradi** – Masalan: fayl serverga yoki RDP orqali ulanish.

---

## 🟡 **Golden Ticket** nima?

### 🎫 Ta'rifi:

Golden Ticket – bu **soxtalashtirilgan TGT** (Ticket Granting Ticket). Ya’ni, siz DC ga o‘zingizni tanishtirmasdan to‘g‘ridan-to‘g‘ri o‘zingiz TGT yaratib olasiz.

### 🔐 Golden Ticket Yaratish Uchun Nima Kerak:

* `KRBTGT` akkauntining **NTLM hash**i (bu DC Sync orqali olinadi)
* **Domain nomi** (masalan: `za.tryhackme.loc`)
* **Domain SID**
* **Foydalanuvchi ID** (odatda 500 – bu `Administrator`)

### 🎯 Afzalliklari:

* DC bilan autentifikatsiya qilish bosqichlari (1 va 2) **butunlay o‘tilib ketiladi**
* Har qanday akkauntni (hatto mavjud bo'lmagan) TGT qilib yaratish mumkin
* TGT’ni 10 yil yoki undan ko‘proq muddatga amal qiladigan qilib qilish mumkin
* **KRBTGT hash** doimiy bo‘lsa, **cheksiz ruxsat** bo’ladi
* DC parolini **ikki marta aylantirmaguncha** bu hujumni to‘xtatish **mushkul**
* Hatto **smart card autentifikatsiyasi**ni ham chetlab o‘tadi
* **Tarmoqdan tashqaridagi kompyuterda ham** ticket yaratish mumkin

---

## ⚪ **Silver Ticket** nima?

### 🎫 Ta'rifi:

Silver Ticket – bu **soxtalashtirilgan TGS** (xizmat uchun ticket). Bunda siz umuman DC bilan bog‘lanmaysiz, faqat kerakli xizmatga soxta ticket yuborasiz.

### 🔐 Silver Ticket Yaratish Uchun Nima Kerak:

* **Xizmat ko‘rsatuvchi kompyuter**ning **NTLM hash’i** (masalan: `THMSERVER1$`)
* **Domain nomi va SID**
* **Foydalanuvchi ID**
* **Xizmat nomi** (`cifs` – bu faylga kirish xizmatidir)
* **Kompyuter nomi** (`THMSERVER1`)

### 🎯 Afzalliklari:

* DC **umuman chaqirilmaydi**, ya’ni **loglar faqat xizmat kompyuterida** qoladi → **Blue Team aniqlay olmaydi**
* Har qanday foydalanuvchini yaratish mumkin, faqat kerakli **SID** ni qo‘shish kerak
* **Mashina akkauntining paroli 30 kunda** o‘zgaradi, lekin bu sozlamani o‘zgartirib **doimiy qilish mumkin**
* Tarmoqdagi boshqa xizmatlarga **faylga, RDP ga va boshqa xizmatlarga** hujum qilish mumkin

---

## ⚔️ Amaliyot: Golden & Silver Ticket Yaratish

### 🔸 Golden Ticket yaratish:

```powershell
kerberos::golden /admin:ReallyNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:S-1-5-21-3885271727-2693558621-2658995185 /krbtgt:<KRBTGT_hash> /endin:600 /renewmax:10080 /ptt
```

✅ So‘ng `dir \\thmdc.za.tryhackme.loc\c$\` buyrug‘i orqali tekshiriladi.

---

### 🔹 Silver Ticket yaratish:

```powershell
kerberos::golden /admin:StillNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:S-1-5-21-3885271727-2693558621-2658995185 /target:THMSERVER1 /rc4:<THMSERVER1_hash> /service:cifs /ptt
```

✅ So‘ng `dir \\thmserver1.za.tryhackme.loc\c$\` orqali tekshiriladi.

---

## 🛡 Nega "Flush all Golden and Silver Tickets!" deyishadi?

Sababi:

* Agar attacker `KRBTGT` yoki kompyuter akkauntining hashini olib bo‘lsa – u **doimiy ruxsatga ega** bo‘ladi.
* **Golden ticket** – *global hujum* (butun domen uchun)
* **Silver ticket** – *xizmatga oid hujum* (aniq hostga)
* Shuning uchun **Blue Team** har doim:

  * KRBTGT parolini **ikki marta aylantirish**,
  * Kompyuter akkauntlarini **tahlil qilish**,
  * Soxta ticketlar aniqlanishi uchun **loglarni monitoring qilish** shart.

---

## ✅ Xulosa:

| Turi              | Tushuntirish                                                   |
| ----------------- | -------------------------------------------------------------- |
| **Golden Ticket** | DC’dan mustaqil ravishda TGT yasab, har qanday xizmatga kirish |
| **Silver Ticket** | DC’ni chetlab o‘tib, faqat ma’lum xizmat uchun kirish          |
| **Asosiy narsa**  | KRBTGT yoki kompyuter akkauntining NTLM hashini olish          |

Agar sizga kerak bo‘lsa, bu amaliyotni o‘zingizning TryHackMe laboratoriyangizda takrorlashingiz mumkin.
