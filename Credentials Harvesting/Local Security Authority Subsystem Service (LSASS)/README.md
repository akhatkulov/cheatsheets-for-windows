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
