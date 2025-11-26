

## **AD obyektlari va enumeratsiyaga o‘tishdan oldin — Credential Injection**

Breaching AD network bo‘limida ko‘rganingizdek, ko‘pincha domen-ga ulangan kompyuterni egallamasdan ham credentiallarni topish mumkin. Ba’zi enumeratsiya usullari ishlashi uchun esa maxsus sharoit kerak bo‘ladi.

---

## **Windows vs Linux**

> “Agar dushmanni ham, o‘zingni ham bilsang — yuz jangning natijasidan qo‘rqmaysan. Faqat o‘zingni bilsang-yu, dushmanni bilmasang — har bir g‘alabadan keyin ham mag‘lubiyat keladi.”
> — Sun Szu, Urush san’ati

Kali Linuxda ham juda uzoq masofaga bora olasiz — AD enumeratsiyasi, tahlil, skanlar… ammo chuqur enumeratsiya va ekspluatatsiya uchun siz **dushmanni tushunishingiz va uni taqlid qilishingiz kerak**.

Ya’ni — **Windows mashinasi** zarur.

Bu bizga ichki, qonuniy Windows vositalaridan foydalanib enumeration va exploitlarni bosqichma-bosqich bajarish imkonini beradi. Bu tarmoqda biz ana shunday vositalardan biri — **runas.exe** bilan ishlaymiz.

---

## **Runas tushuntirildi**

Hech AD credentiallarini topib, lekin ulardan foydalanishga joy yo‘qligini ko‘rganmisiz?

**Runas siz qidirayotgan javob bo‘lishi mumkin!**

Pentest yoki red team baholashlarda sizda tarmoqqa kirish bo‘ladi, credentiallar qo‘lga kiritiladi, lekin yangi domen mashinasi yaratish huquqiga ega bo‘lmaysiz. Shuning uchun credentiallarni **o‘zimiz nazorat qiladigan Windows mashinasida ishlatishimiz kerak**.

Agar credentiallar `<username>:<password>` formatida bo‘lsa, biz **runas.exe** yordamida credentiallarni xotiraga “inject” qilishimiz mumkin:

```
runas.exe /netonly /user:<domain>\<username> cmd.exe
```

### Parametrlar:

✅ **/netonly**
Biz domen-ga ulangan emasmiz, shuning uchun credentiallar faqat **tarmoq autentifikatsiyasi** uchun yuklanadi. Lokal buyruqlar esa kompyuterdagi odatiy foydalanuvchi kontekstida ishlaydi.

✅ **/user**
Bu yerga domen va foydalanuvchi beriladi. Har doim domenning **FQDN** formatidan foydalanish tavsiya qilinadi — bu DNS aniqligini oshiradi.

✅ **cmd.exe**
Credential inject bo‘lgandan keyin ochiladigan dastur. Ko‘pincha cmd eng xavfsiz variant.

Buyruq bajarilgandan so‘ng sizdan parol so‘raladi. `/netonly` ishlatilgani uchun parol domen kontrollerida tekshirilmaydi — har qanday kiritilgan parol qabul qilinadi. Ammo credentiallar tarmoqqa to‘g‘ri yuklanganini tekshirishimiz kerak.

---

### **Muhim eslatma**

Agar o‘z Windowsingizdan foydalansangiz, birinchi Command Prompt’ni **Administrator sifatida** oching. Bu lokal admin tokenini CMD ga qo‘shadi. Bu tarmoqdagi adminlik bermaydi, lekin lokal buyruqlarda kerak bo‘ladi.

---

## **Bu har doim DNS masalasi**

Agar mashinaingizda ichki DNS avtomatik sozlanmagan bo‘lsa, uni qo‘lda sozlashga to‘g‘ri keladi.

DNS server sifatida odatda **Domain Controller IP** ishlatiladi:

PowerShell:

```powershell
$dnsip = "<DC IP>"
$index = Get-NetAdapter -Name 'Ethernet' | Select-Object -ExpandProperty 'ifIndex'
Set-DnsClientServerAddress -InterfaceIndex $index -ServerAddresses $dnsip
```

(`Ethernet` — TryHackMe tarmog‘iga ulangan interfeys nomi)

DNS ishlayotganini tekshiramiz:

```
nslookup za.tryhackme.com
```

Endi credentiallar ishlayotganini test qilish uchun SYSVOL ni listing qilamiz:

```
dir \\za.tryhackme.com\SYSVOL\
```

SYSVOL — barcha Domain Controllerlarda mavjud bo‘lgan papka bo‘lib, unda GPO’lar, domen skriptlari va konfiguratsiyalar saqlanadi.

Ba’zida SYSVOL ichida qo‘shimcha credentiallar ham topiladi, shuning uchun uni albatta enumerate qilish kerak.

---

## **IP va hostname o‘rtasidagi farq**

Savol:
`dir \\za.tryhackme.com\SYSVOL` bilan
`dir \\<DC IP>\SYSVOL`
o‘rtasida farq bormi?

✅ Ha — muhim farq bor, u **autentifikatsiya turi**.

* Hostname orqali — **Kerberos** ishlaydi
* IP orqali — **NTLM** ga majburlanadi

Kerberos biletlari hostname’ni ichiga oladi, shu sababli IP ishlatilganda u ishlamaydi. Pentestda bu juda foydali, chunki:

🔹 ba’zi tashkilotlar Kerberos-based Pass-the-Hash yoki OverPass-hash hujumlarini kuzatadi
🔹 NTLM esa ko‘pincha e’tiborsiz qoladi

Shuning uchun ba’zida IP ishlatish — **stels rejimi** uchun yaxshi taktika.

---

## **Inject qilingan credentiallardan foydalanish**

/netonly bilan ochilgan oynadan bajarilgan **har bir tarmoq so‘rovi** shu credentiallar orqali autentifikatsiya bo‘ladi. Hatto:

✅ MS SQL Studio
✅ NTLM bilan himoyalangan web saytlarga kirish
✅ Windows Authentication talab qiladigan xizmatlar

Masalan, lokal mashinada SQL Server Management Studio ochiladi, foydalanuvchi nomi kompyuter logini bo‘lib tursa ham, login tugmasi bosilganda AD credentiallari ishlaydi.

Bu credential injectionning eng kuchli tomoni.

