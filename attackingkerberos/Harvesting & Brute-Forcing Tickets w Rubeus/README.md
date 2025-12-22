Rubeus - Kerberos hujumlari uchun kuchli vosita. Rubeus, aktiv direktoriya bo‘yicha juda mashhur mutaxassis HarmJ0y tomonidan ishlab chiqilgan va kekeo vositasining moslashuvidir.

Rubeus Kerberosga hujum qilishda juda ko‘p imkoniyatlarga ega bo‘lib, quyidagi hujum va funktsiyalarni o‘z ichiga oladi: overpass the hash, ticket requests and renewals, ticket management, ticket extraction, harvesting, pass the ticket, AS-REP Roasting va Kerberoasting.

Bu vosita juda ko‘p hujum va funksiyalarga ega bo‘lgani uchun men faqat eng muhimlarini tushuntirib o‘taman, lekin siz Rubeusning barcha hujum va imkoniyatlarini o‘rganish uchun quyidagi havolada kengroq ma’lumot olishingiz mumkin: https://github.com/GhostPack/Rubeus

Rubeus allaqachon kompilyatsiya qilingan va nishon mashinasida mavjud.



Rubeus yordamida Ticketlarni Yig‘ish (Harvesting) -

Bu usul KDC ga uzatilayotgan ticketlarni yig‘ib, ularni "pass the ticket" kabi boshqa hujumlarda foydalanish uchun saqlaydi.

1.) cd Downloads - Rubeus joylashgan papkaga o‘ting.

2.) Rubeus.exe harvest /interval:30 - Bu buyruq Rubeus ga har 30 soniyada TGT larni yig‘ishni buyuradi.



Rubeus yordamida Parolni Brut-forcing / Password-Spraying -

Rubeus ham parollarni brut-fors qilishi, ham foydalanuvchi hisoblariga parol "sprеylash" (password spraying) qilishi mumkin. Brut-forsda siz bitta foydalanuvchi hisobiga turli parollarni ro‘yxat bilan sinab ko‘rasiz. Password sprayingda esa siz bitta parolni (masalan, Password1) domen ichidagi barcha foydalanuvchilarga sinab ko‘rasiz, qaysidir birining shu paroldan foydalanishini aniqlash uchun.

Bu hujum berilgan Kerberos paroli bilan barcha topilgan foydalanuvchilarga sinov o‘tkazadi va .kirbi formatidagi ticket beradi. Bu ticket TGT bo‘lib, KDC dan xizmat ticketlarini olish va "pass the ticket" kabi hujumlarda foydalanish mumkin.

Password spraying dan oldin, Windows host fayliga domain controller nomini qo‘shishingiz kerak. Echo buyrug‘i yordamida quyidagicha qo‘shishingiz mumkin:

echo MASHINA_IP CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts

1.) cd Downloads - Rubeus joylashgan papkaga o‘ting.

2.) Rubeus.exe brute /password:Password1 /noticket - Bu berilgan parol bilan barcha topilgan foydalanuvchilarga sinov o‘tkazadi va har bir foydalanuvchi uchun .kirbi TGT beradi.



Ushbu hujumni qo‘llashda ehtiyot bo‘ling, chunki hisob blokirovkasi siyosatiga qarab tarmoqdan bloklanishingiz mumkin.
