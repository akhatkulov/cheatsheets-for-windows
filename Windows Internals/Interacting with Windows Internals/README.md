Windows internals bilan ishlash qiyin ko‘rinsa ham, bu jarayon ancha soddalashtirilgan. Windows internals bilan ishlashning eng qulay va o‘rganilgan usuli – bu **Windows API chaqiruvlari** orqali interfeys qilishdir. Windows API operatsion tizim bilan o‘zaro aloqada bo‘lish uchun **native** (asosiy) funksionallikni taqdim etadi. API o‘z ichiga **Win32 API**ni va kamroq hollarda **Win64 API**ni oladi.

Bu xonada biz faqat Windows internals bilan bog‘liq bo‘lgan bir nechta muhim API chaqiruvlaridan qisqacha foydalanishni ko‘rib chiqamiz. Windows API haqida ko‘proq ma’lumot olish uchun alohida **Windows API xonasiga** qarang.

Ko‘pgina Windows internals komponentlari **fizik apparatlar va xotira** bilan o‘zaro ishlashni talab qiladi.

Windows **yadrosi (kernel)** barcha dasturlar va jarayonlarni boshqaradi hamda dasturiy ta’minot va apparat vositalari o‘rtasida ko‘prik vazifasini bajaradi. Bu juda muhim, chunki ko‘plab Windows internals xotira bilan ishlashni talab qiladi.

Oddiy holda dastur kernel bilan bevosita ishlay olmaydi yoki apparatni o‘zgartira olmaydi va buning uchun maxsus interfeys kerak bo‘ladi. Bu muammo **protsessor rejimlari** va **kirish darajalari** orqali hal qilinadi.

Windows protsessori **user mode** va **kernel mode** rejimlariga ega. Protsessor kirish huquqiga va so‘rov qilinayotgan rejimga qarab ushbu rejimlar o‘rtasida almashadi.

User mode ↔ Kernel mode o‘tish jarayoni ko‘pincha **system call**lar va **API chaqiruvlari** orqali amalga oshadi. Hujjatlarda bu jarayon ba’zida **"Switching Point"** deb ataladi.

| User mode (foydalanuvchi rejimi)                       | Kernel mode (yadro rejimi)                            |
| ------------------------------------------------------ | ----------------------------------------------------- |
| To‘g‘ridan-to‘g‘ri apparatga kirish yo‘q               | To‘g‘ridan-to‘g‘ri apparatga kirish mavjud            |
| Har bir jarayon uchun alohida virtual manzil maydoni   | Barcha jarayonlar uchun umumiy virtual manzil maydoni |
| Faqat "o‘ziga tegishli xotira joylariga" kirish mumkin | Butun fizik xotiraga kirish mumkin                    |

User mode’da yoki “userland”da ishga tushgan dastur shu rejimda qoladi, toki **system call** qilinmaguncha yoki API orqali murojaat qilinmaguncha. System call amalga oshirilganda, dastur rejimni o‘zgartiradi.

Agar dastur tillar orqali **Win32 API** bilan ishlayotgan bo‘lsa, jarayon yanada murakkablashadi. Masalan, **C#** dasturi Win32 API bilan bevosita ishlashdan avval **CLR (Common Language Runtime)** orqali o‘tadi va keyin system call bajaradi.

---

Biz **message box** ni lokal jarayonga in'ektsiya qilib, xotira bilan ishlash bo‘yicha **Proof-of-Concept** (POC) ko‘rsatamiz.

Message box ni xotiraga yozish bosqichlari quyidagicha:

1. Message box uchun lokal jarayon xotirasida joy ajratish.
2. Message box ni ajratilgan xotiraga yozish/kopiyalash.
3. Message box ni lokal jarayon xotirasidan ishga tushirish.

---

**1-bosqich:** Jarayon identificatori (PID) asosida **OpenProcess** orqali jarayon deskriptorini olish:

```c
HANDLE hProcess = OpenProcess(
	PROCESS_ALL_ACCESS, // Kirish huquqlari
	FALSE, // Target handle meros bo‘lmaydi
	DWORD(atoi(argv[1])) // Jarayon ID (komanda qatoridan olinadi)
);
```

---

**2-bosqich:** **VirtualAllocEx** yordamida xotira ajratish:

```c
remoteBuffer = VirtualAllocEx(
	hProcess, // Maqsad jarayon
	NULL, 
	sizeof payload, // Ajratiladigan xotira hajmi
	(MEM_RESERVE | MEM_COMMIT), // Sahifalarni rezerv va commit qilish
	PAGE_EXECUTE_READWRITE // O‘qish/yozish/ishga tushirish ruxsati
);
```

---

**3-bosqich:** **WriteProcessMemory** orqali payloadni xotiraga yozish:

```c
WriteProcessMemory(
	hProcess, // Maqsad jarayon
	remoteBuffer, // Ajratilgan xotira
	payload, // Yoziladigan ma’lumot
	sizeof payload, // Ma’lumot hajmi
	NULL
);
```

---

**4-bosqich:** **CreateRemoteThread** orqali payloadni ishga tushirish:

```c
remoteThread = CreateRemoteThread(
	hProcess, // Maqsad jarayon
	NULL, 
	0, // Default stack hajmi
	(LPTHREAD_START_ROUTINE)remoteBuffer, // Thread boshlanish manzili
	NULL, 
	0, // Yaratilgach darhol ishlaydi
	NULL
); 
```

