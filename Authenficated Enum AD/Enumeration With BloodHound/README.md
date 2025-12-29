**BloodHound** Active Directory (AD) muhitini enumeratsiya qilish uchun eng kuchli vosita bo‘lib qolmoqda va 2016-yildagi chiqarilishidan buyon ushbu sohani tubdan o‘zgartirdi. Dastlab **SpecterOps** tomonidan ishlab chiqilgan ushbu vosita AD xavfsizligiga **grafga asoslangan yondashuv**ni olib kirdi — bu esa hujumchilar va himoyachilar tomonidan keng qabul qilingan fundamental o‘zgarish bo‘ldi.

## Qisqacha tarix

Yillar davomida red team mutaxassislari — va afsuski, hujumchilar — murakkab AD munosabatlarini ko‘rish imkoniyatiga ega edi. BloodHound paydo bo‘lishi bilan xavfsizlik mutaxassislari ruxsatlar, guruh a’zoligi va ishonch munosabatlarini alohida ro‘yxatlar o‘rniga **graf shaklida** ko‘rish imkoniga ega bo‘ldi.

Bu o‘zgarishni quyidagi mashhur ibora juda yaxshi ifodalaydi:

> **“Himoyachilar ro‘yxatlar bilan o‘ylaydi. Hujumchilar esa graflar bilan o‘ylaydi.”** — John Lambert

An’anaviy ravishda himoyachilar Domain Adminlar yoki serverlar ro‘yxati kabi statik ma’lumotlar bilan ishlagan. Hujumchilar esa foydalanuvchilar, guruhlar va kompyuterlar o‘rtasidagi yashirin munosabatlardan foydalangan — bular faqat graf tahlili orqali ko‘rinadigan aloqalardir.

Microsoft ushbu yondashuvning samaradorligini tan olib, keyinchalik **Defender for Identity** ichiga shunga o‘xshash grafga asoslangan metodologiyalarni integratsiya qildi.

## Ikki bosqichli hujum modeli

BloodHound Active Directory muhitlariga qarshi hujum qilishda yangi ikki bosqichli yondashuvni shakllantirdi.

### 1-bosqich: Enumeratsiya

Hujumchilar **SharpHound** yoki **BloodHound-python** kabi kollektorlarni ishga tushirib, AD tuzilmasi haqida ma’lumot yig‘adi. Bu ma’lumotlarga quyidagilar kiradi:

* foydalanuvchi sessiyalari
* guruh a’zoligi
* Access Control List (ACL)
* delegatsiya sozlamalari

Agar blue team hujumni erta aniqlasa ham, hujumchi allaqachon **offline hujum grafigini** qurish uchun yetarli ma’lumotga ega bo‘ladi.

### 2-bosqich: Maqsadli hujum

BloodHound yordamida hujumchilar Domain Admin huquqlarini qo‘lga kiritish kabi aniq va samarali yo‘llarni aniqlaydi. Muhitga qayta kirgach, ular bir necha daqiqa ichida lateral harakat va privilege escalation amalga oshirishi mumkin — ko‘pincha blue team birinchi ogohlantirishga javob berishidan ham tezroq.

Bu imkoniyat BloodHound’ni hujumchilar va proaktiv himoyachilar uchun ajralmas vositaga aylantirdi.

## Rivojlanish va zamonaviy imkoniyatlar

BloodHound vaqt o‘tishi bilan sezilarli darajada rivojlandi:

* **Azure qo‘llab-quvvatlashi**: AzureHound yordamida Azure Entra ID va on-prem AD muhitlari enumeratsiya qilinadi
* **Yangi hujum primitivlari**: Resource-Based Constrained Delegation (RBCD) kabi texnikalar uchun `AddAllowedToAct`, `AllowedToAct` primitivlari qo‘shildi
* **Ilg‘or tahlil algoritmlari**: Butterfly algoritmi xavf darajasini baholash va ustuvorlikni aniqlashda yordam beradi

Bu yangilanishlar BloodHound’ga katta va gibrid (bulut + on-prem) muhitlarni chuqur tahlil qilish imkonini beradi.

## Himoyachilar nuqtai nazari

Bugungi kunda himoyachilar **BloodHound** va **BloodHound Enterprise** yordamida noto‘g‘ri sozlamalar, ortiqcha huquqlar va xavfli kirish yo‘llarini hujumchilar ekspluatatsiya qilishidan oldin aniqlaydi.

AD munosabatlarini doimiy ravishda xaritalash va monitoring qilish orqali tashkilotlar o‘z infratuzilmasini mustahkamlaydi.

## Ma’lumot yig‘ish

Ko‘pincha **SharpHound** va **BloodHound-python** bir xil vosita sifatida tilga olinadi, ammo ular bir xil emas.

**SharpHound** — bu BloodHound’ning rasmiy **ma’lumot yig‘uvchi komponenti** bo‘lib, AD ma’lumotlarini yig‘adi. BloodHound esa ushbu ma’lumotlarni graf ko‘rinishida vizuallashtiradi.

## SharpHound’ni tushunish

SharpHound C# tilida yozilgan va quyidagilarni enumeratsiya qiladi:

* guruh a’zoligi
* sessiya ma’lumotlari
* ACL’lar
* domen ishonchlari
* lokal administrator huquqlari

### SharpHound kollektor turlari

* **SharpHound.exe** – Windows tizimlari uchun tavsiya etilgan asosiy kollektor
* **AzureHound.ps1** – Azure Entra ID muhitlari uchun PowerShell skripti
* **SharpHound.ps1 (eskirgan)** – ilgari stealth maqsadlarda ishlatilgan, hozir qo‘llab-quvvatlanmaydi

## BloodHound.py (Python kollektor)

Linux muhitlar uchun mos keladi. U quyidagi autentifikatsiya usullarini qo‘llab-quvvatlaydi:

* parol
* NTLM hash
* Kerberos ticket

Natijalar BloodHound GUI’ga mos ZIP yoki JSON formatida chiqariladi.

> **Eslatma:** BloodHound va SharpHound versiyalari mos bo‘lishi kerak. BloodHound jamoasi BloodHound.py’ni rasmiy qo‘llab-quvvatlamaydi.

## SharpHound’ni ishga tushirish

Windows domeniga ulangan mashinada quyidagi buyruq ishlatiladi:

```
SharpHound.exe --CollectionMethods All --Domain tryhackme.loc --ExcludeDCs
```

Parametrlar:

* `--CollectionMethods All` – barcha usullar
* `--Domain` – maqsad domen
* `--ExcludeDCs` – Domain Controller’larni chiqarib tashlash

## BloodHound.py’dan foydalanish

Linux muhitida:

```
bloodhound-python -u asrepuser1 -p qwerty123! -d tryhackme.loc -ns 10.211.12.10 -c All --zip
```

Bu buyruq ZIP fayl hosil qiladi va uni BloodHound’ga import qilish mumkin.

## Operatsion xavfsizlik (OPSEC)

* `--ExcludeDCs` ishlating
* DCOnly kabi kam shovqinli metodlardan foydalaning
* Antivirus istisnolari yoki `/netonly` bilan `runas` ishlating
* Har doim ruxsat bilan ishlang

## BloodHound-CE’ga kirish

* Brauzerda `http://10.211.12.100:8080`
* Login: `admin`
* Parol: `weU^BjZr33OIWsC^`

## Ma’lumotlarni import qilish

* **Administration** → **File Ingest**
* ZIP faylni yuklang

## BloodHound ma’lumotlarini o‘rganish

* **Explore** bo‘limi orqali grafni ko‘ring
* Node — obyektlar
* Edge — munosabatlar

## Built-in so‘rovlar

* **Cypher** → tayyor so‘rovlar
* Masalan: “All Domain Admins”

## Attack Path topish

* **Pathfinding** tugmasi
* Boshlanish va maqsad node’ni tanlang
* Agar yo‘l mavjud bo‘lsa, u vizual tarzda ko‘rsatiladi

## Afzalliklar va cheklovlar

**Afzalliklar:**

* Web-interfeys
* Aniq attack path’lar
* AD munosabatlarini chuqur tahlil

**Cheklovlar:**

* SharpHound shovqinli
* AV/EDR trigger bo‘lishi mumkin
