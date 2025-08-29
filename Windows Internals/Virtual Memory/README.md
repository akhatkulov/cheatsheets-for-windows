
**Virtual xotira** – bu Windows ichki tuzilmalari qanday ishlashi va bir-biri bilan qanday o‘zaro aloqada bo‘lishining muhim komponentidir. Virtual xotira boshqa ichki komponentlarga xuddi jismoniy xotira kabi murojaat qilish imkonini beradi, ammo ilovalar o‘rtasida to‘qnashuv (collision) xavfisiz. **Rejimlar va to‘qnashuvlar** tushunchasi 8-vazifada batafsil tushuntiriladi.

Virtual xotira har bir jarayonga **shaxsiy virtual manzil maydonini** taqdim etadi. **Xotira boshqaruvchisi (memory manager)** virtual manzillarni jismoniy manzillarga tarjima qilish uchun ishlatiladi. Shaxsiy virtual manzil maydoni mavjudligi va jismoniy xotiraga bevosita yozilmasligi sababli, jarayonlar zarar yetkazish xavfini kamaytiradi.

Xotira boshqaruvchisi shuningdek **sahifalar (pages)** yoki **ko‘chirishlar (transfers)** orqali xotirani boshqaradi. Ilovalar ajratilgan jismoniy xotiradan ko‘proq virtual xotiradan foydalanishi mumkin; bu muammoni hal qilish uchun xotira boshqaruvchisi virtual xotirani diskka o‘tkazadi yoki sahifalaydi. Bu tushunchani quyidagi diagrammada tasavvur qilish mumkin.

![](https://raw.githubusercontent.com/akhatkulov/cheatsheets-for-windows/refs/heads/main/Windows%20Internals/Virtual%20Memory/windowsinternals-1.png)
![](https://raw.githubusercontent.com/akhatkulov/cheatsheets-for-windows/refs/heads/main/Windows%20Internals/Virtual%20Memory/windowsinternals-2.png)

* **32-bit x86 tizimida** nazariy maksimal virtual manzil maydoni **4 GB**.

  * Bu manzil maydoni ikkiga bo‘lingan:

    * **Quyi yarmi** (0x00000000 – 0x7FFFFFFF) jarayonlarga ajratiladi.
    * **Yuqori yarmi** (0x80000000 – 0xFFFFFFFF) esa operatsion tizimning xotira foydalanishiga ajratiladi.
  * Administratorlar ushbu taqsimotni **(increaseUserVA sozlamasi)** yoki **AWE (Address Windowing Extensions)** orqali o‘zgartirib, ko‘proq manzil maydoni talab qiladigan ilovalar uchun kengaytirishlari mumkin.

* **64-bit zamonaviy tizimlarda** nazariy maksimal virtual manzil maydoni **256 TB**.

  * 32-bit tizimidagi manzil maydoni taqsimoti nisbati 64-bit tizimida ham qo‘llaniladi.
  * Amalda, ko‘pgina muammolar (sozlamalar yoki AWE talab qiladigan) bu kengaytirilgan maksimal hajm orqali hal qilinadi.

