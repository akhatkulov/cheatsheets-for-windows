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

![NTLM]()
