

## ❖ Bloodhound orqali AD Enumeration

Bloodhound — hozirgacha mavjud bo‘lgan **eng kuchli Active Directory enum vositasi**. 2016-yilda chiqarilganda, AD enumeratsiya jarayonini tubdan o‘zgartirdi.

### 📌 Bloodhound tarixi

Uzoq vaqt davomida red teamchilar (va, afsuski, haqiqiy hujumchilar) ustunlikka ega bo‘lishgan. Holat shu darajada ediki, Microsoft o‘zining Advanced Threat Protection tizimiga Bloodhound’ga o‘xshash funksiyani qo‘shishga majbur bo‘ldi.

Buning asosiy sababi quyidagi mashhur ibora edi:

> **"Himoyachilar ro‘yxatlarda o‘ylashadi, hujumchilar esa graflarda."**

Bloodhound hujumchilarga AD infratuzilmasini **graf ko‘rinishida**, bog‘langan tugunlar (nodes) bilan tasvirlash imkonini berdi. Har bir bog‘lanish — hujum uchun potentsial yo‘l.
Blue Team esa odatda shunchaki ro‘yxatlardan foydalanardi — masalan, Domain Admins ro‘yxati, hostlar ro‘yxati va h.k.

Grafik tafakkur hujumchilarga yangi daraja ochib berdi. Hujum ikki bosqichga bo‘lindi:

1. **Birlamchi phishing** — tarmoqqa kirish va AD ma’lumotlarini yig‘ish
   (odatda shovqinli, aniqlanadi, to‘xtatiladi)

2. **Offline tahlil** — yig‘ilgan ma’lumotlardan hujum yo‘lini qurish
   (graf ko‘rinishida aniq hujum marshrutini topish)

So‘ng yana kichik phishing yoki ekspluatatsiya bilan hujumchilar **bir necha daqiqada maqsadga yetishlari** mumkin edi — Blue Team signal olguncha.

Shuning uchun bugun ko‘plab Blue Teamlar ham Bloodhound’dan foydalanadi — xavfsizlik holatini yaxshiroq tushunish uchun.

---

## ❖ Sharphound nima?

Ko‘p odam Bloodhound va Sharphound nomlarini almashtirib ishlatadi — lekin ular **bir xil emas**.

✅ **Sharphound** — AD ma’lumotlarini yig‘uvchi collector
✅ **Bloodhound** — ushbu ma’lumotlarni vizual tahlil qiluvchi GUI

Demak, avval Sharphound bilan ma’lumot to‘playmiz — keyin uni Bloodhound’ga yuklaymiz.

### Sharphound’ning 3 turi bor:

1. **Sharphound.ps1** — PowerShell script

   * RAT ichida, memory execution uchun qulay
   * Diskka yozilmagani sabab AVdan qochishi oson
   * Ammo yangi versiyalarda chiqarilishi to‘xtatilgan

2. **Sharphound.exe** — Windows executable

   * Eng ko‘p ishlatiladi

3. **AzureHound.ps1**

   * Azure AD enumeratsiyasi uchun

📌 Muhim: **Sharphound va Bloodhound versiyalari mos bo‘lishi kerak**, aks holda ZIP import qilinmaydi. Bu tarmoq uchun Bloodhound **v4.1.0** talab qilinadi.

---

## ❖ Sharphound qachon aniqlanadi?

Collector fayllar ko‘pincha:

* Antivirus
* EDR
* SOC monitoring

tomonidan **malware sifatida flag qilinadi**.

Shuning uchun red team ko‘pincha:

✅ Domain-join bo‘lmagan Windows mashinadan
✅ `runas` orqali AD credential kiritib
✅ AV’ni o‘chirib yoki exclusion qo‘yib

Sharphound’ni ishga tushiradi.

TryHackMe’dagi THMJMP1 mashinasida bu allaqachon bajarilgan, va `C:\Tools\` papkasida Sharphound mavjud.

---

## ❖ Sharphoundni ishga tushirish

PowerShell oching va faylni Documents’ga ko‘chiring:

```
copy C:\Tools\Sharphound.exe ~\Documents\
cd ~\Documents\
```

So‘ng ishga tushiring:

```
Sharphound.exe --CollectionMethods All --Domain za.tryhackme.com --ExcludeDCs
```

### Parametrlar:

* **CollectionMethods**

  * qanday ma’lumot yig‘iladi (Default, All, Session)

* **Domain**

  * qaysi domenni enumeratsiya qilish

* **ExcludeDCs**

  * domain kontrollerlariga tegmaslik
  * aniqlanish riskini kamaytiradi

Jarayon 1 daqiqa davom etadi va yakunda ZIP fayl yaratiladi.

---

## ❖ Bloodhoundga yuklash

Bloodhound Neo4j graf bazasidan foydalanadi.

1. Neo4j’ni ishga tushiring:

```
neo4j console start
```

2. Bloodhound’ni ishga tushiring:

```
bloodhound --no-sandbox
```

Default Neo4j login:

```
neo4j / neo4j
```

3. ZIP faylni Windowsdan olib keling:

```
scp <user>@THMJMP1.za.tryhackme.com:C:/Users/<user>/Documents/<zip> .
```

4. ZIPni Bloodhound oynasiga drag & drop qiling.

Import tugagach — graf tahliliga tayyorsiz ✅

---

## ❖ Bloodhounddagi asosiy imkoniyatlar

Node’ni tanlaganda quyidagi bo‘limlar chiqadi:

* **Overview**
* **Node Properties**
* **Extra Properties**
* **Group Membership**
* **Local Admin Rights**
* **Execution Rights**
* **Outbound/Inbound Control Rights**

Masalan, Group Membership bo‘limi — user qaysi guruhlarga a’zo ekanini ko‘rsatadi.

---

## ❖ Attack Path topish

Bloodhoundning eng kuchli funksiyasi — hujum yo‘lini avtomatik topish.

Masalan:

**Start:** bizning AD user
**End:** Tier 1 Admins

Agar yo‘l mavjud bo‘lsa — Bloodhound graf ko‘rinishida ko‘rsatadi.

Bu yo‘lda:

* session misconfigurations
* broken tiering model
* RDP access
* ACL huquqlari
* delegation
* nested guruhlar

kabi zaifliklardan foydalaniladi.

---

## ❖ Session ma’lumotlari nega muhim?

ADning struktura qismi ko‘p o‘zgarmaydi.

Lekin:

✅ user sessiyalari
✅ logonlar
✅ endpointlar

har doim o‘zgaradi.

Shuning uchun:

* tahlil boshida *All* bilan ishga tushiring
* kun davomida bir necha marta *Session* bilan

Bloodhoundda eskirgan sessionlarni **Clear Session Information** bilan tozalash mumkin.

---

## ✅ Bloodhoundning afzalliklari

* GUI bilan qulay vizual tahlil
* Hujum yo‘llarini topib beradi
* Murakkab AD ma’lumotlarini tez topadi
* Red va Blue Team uchun foydali
* Pentestni tezlashtiradi

## ⚠️ Kamchiliklari

* Sharphound ishga tushishi shovqinli
* Ko‘pincha AV/EDR tomonidan aniqlanadi
* Neo4j o‘rnatilishi kerak
* Analiz uchun tajriba talab qiladi

