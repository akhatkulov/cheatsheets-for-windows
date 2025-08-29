

**Ip (Thread)** – bu jarayon tomonidan ishlatiladigan va qurilma omillariga (device factors) asoslanib rejalashtiriladigan bajariladigan birlikdir.

Qurilma omillari protsessor (CPU) va xotira xususiyatlari, ustuvorlik (priority) va mantiqiy omillar hamda boshqa narsalarga qarab farq qilishi mumkin.

Ipni soddalashtirib ta’riflasak: **"jarayon bajarilishini boshqaruvchi element"**.

Iplar bajarilishni boshqarishi sababli ular tez-tez nishonga olinadigan komponentdir. Ipni suiiste’mol qilish (thread abuse) mustaqil ravishda kodni bajarishda qo‘llanilishi mumkin, biroq ko‘proq hollarda boshqa API chaqiruvlari bilan birlashtirib, turli usullarning bir qismi sifatida ishlatiladi.

Iplar o‘z ota-jarayonining barcha tafsilotlari va resurslarini (kod, global o‘zgaruvchilar va boshqalar) ulashadi. Shu bilan birga, iplarning o‘ziga xos noyob qiymatlari va ma’lumotlari mavjud. Ular quyidagi jadvalda keltirilgan:

| **Komponent**            | **Vazifasi**                                                                        |
| ------------------------ | ----------------------------------------------------------------------------------- |
| **Stack**                | Ipga tegishli barcha ma’lumotlar (istisnolar, protsedura chaqiriqlari va boshqalar) |
| **Thread Local Storage** | Unikal ma’lumotlar muhitini ajratish uchun ko‘rsatkichlar                           |
| **Stack Argument**       | Har bir ipga ajratiladigan noyob qiymat                                             |
| **Context Structure**    | Yadro (kernel) tomonidan saqlanadigan mashina registrlari qiymatlari                |

Iplar sodda va minimal ko‘rinadigan komponentdek tuyulishi mumkin, ammo ularning funksiyasi jarayonlar uchun **nihoyatda muhim**.

