Kerbrute - bu Active Directory hisoblarini Kerberos oldindan autentifikatsiyadan foydalanib sanash va parollarni buzish uchun mo‘ljallangan mashhur dastur . Dastur tez va nisbatan sezilmas ishlashi tufayli, hujumchi tomonidan foydalaniladigan muhim vositalardan biriga aylangan, chunki Kerberos bo‘yicha muvaffaqiyatsiz autentifikatsiya Windowsning standart 4625-xatosi (hisob muvaffaqiyatsiz tizimga kirishga urindi) keltirib chiqarmaydi .

### 🛠️ Kerbrute'ni O'rnatish va Undan Foydalanish

O‘rnatish jarayoni juda sodda. Mana o‘zbek tilidagi bosqichma-bosqich ko‘rsatma:

1.  **Kerbrute'ni yuklab oling**: GitHub'ning rasmiy relizlar sahifasidan operatsion tizimingizga mos keladigan versiyani yuklab oling. `kerbrute_linux_amd64` - Linux tizimlari uchun .
2.  **Faylni nomini o‘zgartiring va ishga tushiruvchi qiling**:
    ```bash
    mv kerbrute_linux_amd64 kerbrute
    chmod +x kerbrute
    ```
3.  **Kerbrute'ni ishga tushirish imkoniyatini kengaytiring** (ixtiyoriy): Faylni `PATH` tizim o‘zgaruvchisiga qo‘shishingiz mumkin, masalan, `/usr/local/bin` papkasiga ko‘chirib, undan keyin terminalda qayerdan bo‘lmasin `kerbrute` deb ishga tushirishingiz mumkin .

Kerbrute bir nechta asosiy buyruqlardan iborat :
| Buyruq | Vazifasi | Xavf (Hisob bloklanishi) |
| :--- | :--- | :--- |
| `userenum` | Domendagi amaldagi foydalanuvchilarni ro‘yxatdan o‘tkazish | **Past**. Hisob bloklanmaydi . |
| `passwordspray` | Bitta parolni bir nechta foydalanuvchilar orqali sinab ko‘rish | **Yuqori**. Bloklanish xavfi bor . |
| `bruteuser` | Bitta foydalanuvchi uchun parollar ro‘yxatini sinab ko‘rish | **Yuqori**. Bloklanish xavfi bor . |
| `bruteforce` | "foydalanuvchi:parol" juftligidan iborat ro‘yxatni sinab ko‘rish | **Yuqori**. Bloklanish xavfi bor . |

### 🔍 Amalda Foydalanuvchilarni Aniqlash (`userenum`)

Kerbrute’ning eng ko‘p ishlatiladigan funksiyasi bu `userenum`. Bu sizda bor foydalanuvchilar ro‘yxatidan haqiqatan domen bo‘yicha mavjud bo‘lgan foydalanuvchilarni aniq aniqlab beradi.

Siz bergan matndan foydalanib, buyruq quyidagicha ko‘rinishi kerak:
```bash
./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt
```

**Muhim izohlar**:
*   Domen va Domen Kontroller nomi sifatida `CONTROLLER.local` dan foydalanishga ishonch hosil qiling. Odatda bu `CONTROLLER.local` bo‘lmaydi. Odatda domen nomi ikki qismdan iborat bo‘ladi (masalan, `company.local`).
*   Domenning IP manzilini aniqlash uchun `/etc/hosts` faylini tahrirlashingiz kerak: `MACHINE_IP CONTROLLER.local` . Buyrukda `--dc CONTROLLER.local` qo‘llanilgan, shuning uchun bu juda muhim.
*   Agar foydalanuvchi mavjud bo‘lsa, lekin siz kiritgan nom bilan kerakli kalit yaratilmasa (masalan, `jDoe` o‘rniga `jdoe`), Kerbrute aniq foydalanuvchi nomini chiqaradi .

Amaliy misol:
```bash
./kerbrute userenum -d ignite.local --dc 192.168.1.19 users.txt -o result.txt
```

### ⚠️ Xavflar va Mudofaa Choralari

Kerbrute faqat huquqlangan va nazorat qilinadigan muayyan muhitlarda (masalan, CTF, o‘quv laboratoriyalari yida rozilik asosida pentest) qo‘llanilishi kerak. Amalda foydalanish quyidagi xavflarga olib keladi :
*   **Huquqbuzarlik**: Kerbrute hakkerlik asbobi (HackTool) sifatida tasniflanadi va noqonuniy foydalanish qonuniy jazolarga olib kelishi mumkin.
*   **Tizimga zarar etkazish**: `passwordspray` va `bruteuser` kabi buyruqlar hisobni bloklab qo‘yishi mumkin, bu haqiqiy tarmoqqa zarar etkazishi mumkin.

**Qanday aniqlash va himoya qilish mumkin**:
*   **Monitoring**: Kerbrute Kerberos orqali hujum qiladi va Event ID **4768** (TGT so‘raldi) va **4771** (Oldin autentifikatsiya muvaffaqiyatsiz) keltirib chiqarishi mumkin. Ko‘p miqdordagi bunday xatolar Kerbrute hujumidan dalolat berishi mumkin .
*   **Parol siyosati**: Murakkab parollar va hisob bloklanish siyosati (`passwordspray` hujumlarini bir qadar samarasiz qiladi) muhim himoya choralaridir .
*   **Audit**: Oldin autentifikatsiyani o‘chirib qo‘ygan hisoblarni muntazam tekshirib borish kerak, chunki ular Kerbrute hujumi uchun zaif hisoblanadi .
