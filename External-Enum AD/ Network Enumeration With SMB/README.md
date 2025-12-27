
Ushbu topshiriqda biz korporativ tarmoq perimetrini buzib o‘tganimizni va Linux asosidagi tizimga kirish huquqiga ega ekanimizni faraz qilamiz. Biz **AttackBox**’dan foydalanamiz, biroq xohlasangiz VPN orqali o‘zingiz yoqtirgan hujumga mo‘ljallangan Linux distributivini ham ulashingiz mumkin.

Biz **Server Message Block (SMB)** protokoli yordamida tarmoqdagi umumiy resurslarni (network shares) enumeratsiya qilishga e’tibor qaratamiz. Buning uchun Nmap kabi turli vositalardan foydalanib, qaysi portlar ochiq ekanini va qanday servislar ishlayotganini aniqlaymiz. Keyin esa **smbclient** va **smbmap** kabi vositalar yordamida AttackBox’dan ularning ichki tarkibiga kirishga harakat qilamiz. Shuningdek, ochiq bo‘lgan SMB share’lar ichidagi fayllarni yuklab olishga urinib ko‘ramiz. Oxirida yana bir nechta foydali vositalarni ham eslatib o‘tamiz. Keling, boshlaymiz!

---

## Servislarni aniqlash (Discovering Services)

Biz “discovery” jarayonini eski va ishonchli **Nmap** bilan boshlaymiz. Asosan MS Windows va Active Directory bilan bog‘liq portlar bizni qiziqtiradi. Skan qilishni quyidagi portlar bilan cheklaymiz:

* **TCP 88 (Kerberos)**
  Kerberos Active Directory’da autentifikatsiya uchun ishlatiladi. Pentest nuqtai nazaridan bu Pass-the-Ticket va Kerberoasting kabi ticket hujumlari uchun juda qimmatli bo‘lishi mumkin.

* **TCP 135 (RPC Endpoint Mapper)**
  Bu port Remote Procedure Call (RPC) uchun ishlatiladi. U lateral harakat (lateral movement) yoki DCOM orqali masofadan kod bajarish imkoniyatlarini aniqlashda foydali bo‘lishi mumkin.

* **TCP 139 (NetBIOS Session Service)**
  Eski Windows tizimlarida fayl almashish uchun ishlatiladi. Null session va ma’lumot yig‘ish uchun suiiste’mol qilinishi mumkin.

* **TCP 389 (LDAP)**
  Lightweight Directory Access Protocol (LDAP) uchun ishlatiladi. Bu port shifrlanmagan bo‘lib, AD obyektlari, foydalanuvchilar va siyosatlarni enumeratsiya qilish uchun asosiy nishon hisoblanadi.

* **TCP 445 (SMB)**
  Fayl almashish va masofaviy administratsiya uchun juda muhim. EternalBlue, SMB relay hujumlari va credential o‘g‘irlash kabi ekspluatatsiyalar uchun ko‘p ishlatiladi.

* **TCP 636 (LDAPS)**
  Secure LDAP uchun ishlatiladi. Shifrlangan bo‘lsa ham, noto‘g‘ri sozlangan bo‘lsa AD tuzilmasini oshkor qilishi va AD CS (sertifikatga asoslangan) hujumlari uchun imkon berishi mumkin.

Biz Nmap yordamida ushbu portlarda servislar ishlayaptimi yoki yo‘qligini tekshiramiz, `-sV` bilan versiyalarni aniqlaymiz va `-sC` orqali default skriptlarni ishga tushiramiz. Yakuniy buyruq quyidagicha bo‘ladi:

```
nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP
```

Quyida domain controller’ni skan qilish natijasi ko‘rsatilgan:

### AttackBox Terminal

```
root@tryhackme:~# nmap -p 88,135,139,389,445,636 -sV -sC 10.211.11.10
[...]
PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-05-15 12:41:17Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
636/tcp open  tcpwrapped
[...]
```

Yuqoridagi ochiq portlar MS Windows Active Directory muhitini ko‘rsatmoqda. Kerberos va LDAP portlarining mavjudligi bu tizim Active Directory domenining bir qismi yoki hatto Domain Controller ekanidan dalolat beradi.

---

## SMB Share’larni ro‘yxatlash (Listing SMB Shares)

Hozircha bizda to‘g‘ri login ma’lumotlari yo‘q. Keling, ochiq SMB share’larni tekshirib, ularga anonim tarzda ulanish mumkinmi-yo‘qligini ko‘ramiz. Bu **null session** deb ataladi, chunki unda foydalanuvchi nomi va parol ishlatilmaydi.

Linux tizimidan SMB share’larni enumeratsiya qilish uchun ikkita juda yaxshi vosita mavjud: **smbclient** va **smbmap**.

### smbclient

`smbclient` — bu Samba paketiga kiruvchi buyruq qatori vositasi bo‘lib, SMB share’lar bilan ishlash imkonini beradi. U FTP klientiga o‘xshaydi: fayllarni ko‘rish, yuklash va yuklab olish mumkin.

Quyidagi terminalda biz `-L` opsiyasi yordamida share’larni ro‘yxatlaymiz va `-N` bilan parolsiz (anonymous) ulanamiz:

### AttackBox Terminal

```
root@tryhackme:~# smbclient -L //10.211.11.10 -N
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        AnonShare       Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SharedFiles     Disk      
        SYSVOL          Disk      Logon server share 
        UserBackups     Disk      
SMB1 disabled -- no workgroup available
```

---

### smbmap

`smbmap` — bu SMB share’larni tezkor enumeratsiya qilish uchun mo‘ljallangan recon vositasi. U har bir share uchun READ va WRITE ruxsatlarini ko‘rsatadi va noto‘g‘ri sozlangan share’larni tezda aniqlashga yordam beradi.

Quyida `smbmap -H TARGET_IP` buyrug‘i misoli keltirilgan. E’tibor bering, AttackBox’da smbmap quyidagi joyda joylashgan:

`/root/Desktop/Tools/Miscellaneous/smbmap`

### AttackBox Terminal

```
root@tryhackme:~/Desktop/Tools/Miscellaneous/smbmap# ./smbmap.py -H 10.211.11.10
[+] Finding open SMB ports....
[+] User SMB session established on 10.211.11.10...
[+] IP: 10.211.11.10:445        Name: 10.211.11.10        
        Disk                     Permissions     Comment
        ----                     -----------     -------
        ADMIN$                   NO ACCESS       Remote Admin
        AnonShare                READ, WRITE
        C$                       NO ACCESS       Default share
        IPC$                     NO ACCESS       Remote IPC
        NETLOGON                 NO ACCESS       Logon server share 
        SharedFiles              READ, WRITE
        SYSVOL                   NO ACCESS       Logon server share 
        UserBackups              READ, WRITE
```

Ushbu buyruqlardan ko‘rinib turibdiki, uchta nostandart share bizning e’tiborimizni tortadi:
**AnonShare**, **SharedFiles** va **UserBackups**.

Shuni ham ta’kidlash kerakki, SMB share’larni **Nmap** yordamida ham aniqlash mumkin. Masalan, `smb-enum-shares` skripti orqali qaysi share’larda READ/WRITE, faqat READ yoki umuman ruxsat yo‘qligini bilib olish mumkin:

```
nmap -p445 --script smb-enum-shares 10.211.11.10
```

---

## SMB Share’larga kirish (Accessing SMB Shares)

Endi share’larni aniqladik, keling, anonim kirish mumkin bo‘lganlariga ulanib ko‘raylik. Balki bizga yordam beradigan qiziqarli fayllar toparmiz.

`smbclient` yordamida share’ga ulanish quyidagicha amalga oshiriladi:

```
smbclient //TARGET_IP/SHARE_NAME -N
```

Ulanganimizdan so‘ng `ls` buyrug‘i bilan fayllarni ko‘ramiz. Agar kerakli faylni topsak, `get file_name` bilan uni AttackBox’ga yuklab olishimiz mumkin.

Quyidagi misolda biz `Mouse_and_Malware.txt` faylini yuklab oldik:

### AttackBox Terminal

```
root@tryhackme:~# smbclient //10.211.11.10/SharedFiles -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu May 15 16:03:40 2025
  ..                                  D        0  Thu May 15 16:03:40 2025
  Mouse_and_Malware.txt               A     1141  Thu May 15 10:40:19 2025

                7863807 blocks of size 4096. 3459432 blocks available
smb: \> get Mouse_and_Malware.txt
getting file \Mouse_and_Malware.txt of size 1141 as Mouse_and_Malware.txt (69.6 KiloBytes/sec)
smb: \> exit
```

Agar sizda foydalanuvchi nomi va parol bo‘lsa, ularni quyidagicha ko‘rsatishingiz mumkin:

* `--user=USERNAME --password=PASSWORD`
* yoki `-U 'username%password'`

Agar domen akkaunti bo‘lsa, domen nomini `-W` opsiyasi bilan ko‘rsatish kerak.

---

## SMB Share’larda nimalarni topish mumkin?

Anonim SMB share — bu juda yomon g‘oya. Biroq ba’zi administratorlar eski tizimlar ishlashi uchun buni yoqib qo‘yishlari mumkin. Masalan, eski printer fayl o‘qishi yoki eski skaner fayl yozishi kerak bo‘lishi mumkin.

Pentester nuqtai nazaridan qaraganda, SMB share’larda quyidagilar bo‘lishi mumkin:

* konfiguratsiya fayllari
* zaxira (backup) fayllari
* skriptlar
* hujjatlar

Ko‘pincha foydalanuvchilar qulaylik uchun fayllarni umumiy papkalarga joylaydi. Agar yozish (WRITE) huquqi bo‘lsa, bu yanada ko‘proq fayllar yuklanishiga olib keladi. Ayrim fayllarda foydalanuvchi nomlari yoki hatto o‘zgartirilmagan parollar bo‘lishi mumkin.

Xulosa qilib aytganda, mavjud barcha share’larni sinchkovlik bilan tekshirish juda muhim, chunki ular yashirin va foydali sirlarni oshkor qilishi mumkin.

---

## Boshqa vositalar va resurslar

Yana bir nechta foydali vositalar mavjud:

* **impacket-smbclient** — Impacket toolkit’iga kiruvchi, smbclient’ning Python asosidagi versiyasi. AttackBox’da u `/opt/impacket/examples/` katalogida joylashgan.

* **CrackMapExec** — nafaqat post-exploitation, balki enumeratsiya uchun ham juda qulay. U SMB share’larni ro‘yxatlash, credential tekshirish va boshqa ko‘plab modullarga ega.

* **enum4linux / enum4linux-ng** — SMB orqali keng qamrovli enumeratsiya qiluvchi kuchli vosita.
  Masalan:

  ```
  enum4linux -a TARGET_IP
  ```

  Natijani faylga yo‘naltirib, keyin sekinlik bilan ko‘rib chiqish tavsiya etiladi.

Va, albatta, oldin aytib o‘tilganidek, **Nmap**’ni `smb-enum-shares` skripti bilan ishlatishni ham unutmasligimiz kerak.


