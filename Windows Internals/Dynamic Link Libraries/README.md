Microsoft hujjatlarida DLL shunday ta’riflanadi: **“Bir vaqtning o‘zida bir nechta dastur tomonidan ishlatilishi mumkin bo‘lgan kod va ma’lumotlarni o‘z ichiga olgan kutubxona.”**

DLL’lar Windows’da dastur bajarilishining asosiy funksiyalaridan biri sifatida qo‘llaniladi. Windows hujjatlarida keltirilishicha:
**“DLL’lardan foydalanish kodni modullashtirish, qayta ishlatish, xotirani samarali ishlatish va diskda joyni tejashni ta’minlaydi. Shunday qilib, operatsion tizim va dasturlar tezroq yuklanadi, tezroq ishlaydi va kompyuterda kamroq joy egallaydi.”**

Dasturda DLL funksiya sifatida yuklanganda, DLL **bog‘liqlik (dependency)** sifatida belgilanadi. Chunki dastur DLL’ga bog‘liq bo‘ladi, hujumchilar dasturlarning o‘ziga emas, balki DLL’lariga hujum qilib, bajarilish jarayonini yoki funksionallikni nazorat qilishlari mumkin.

DLL bilan bog‘liq hujum texnikalari:

* **DLL Hijacking (T1574.001)**
* **DLL Side-Loading (T1574.002)**
* **DLL Injection (T1055.001)**

DLL yaratish oddiy loyiha yoki ilova yaratishdan farq qilmaydi; faqat kichik sintaksis o‘zgarishlari talab qilinadi. Quyida Visual C++ Win32 Dynamic-Link Library loyihasida yaratilgan DLL misoli keltirilgan:

```cpp
#include "stdafx.h"
#define EXPORTING_DLL
#include "sampleDLL.h"
BOOL APIENTRY DllMain( HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved )
{
    return TRUE;
}

void HelloWorld()
{
    MessageBox( NULL, TEXT("Hello World"), TEXT("In a DLL"), MB_OK);
}
```

Quyida esa DLL uchun **header file** (sarlavha fayli) berilgan; u qanday funksiyalar import va export qilinishini belgilaydi. Biz keyingi bo‘limda header faylning ahamiyatini (yoki unchalik zarurligini) muhokama qilamiz.

```cpp
#ifndef INDLL_H
    #define INDLL_H
    #ifdef EXPORTING_DLL
        extern __declspec(dllexport) void HelloWorld();
    #else
        extern __declspec(dllimport) void HelloWorld();
    #endif
#endif
```

DLL yaratilgach, savol tug‘iladi: ular qanday qilib dasturda ishlatiladi?

DLL’larni dasturga yuklash ikki usulda amalga oshiriladi:

1. **Load-time dynamic linking** (yuklash vaqtida dinamik bog‘lash)
2. **Run-time dynamic linking** (ishlash vaqtida dinamik bog‘lash)

### 1. Load-time dynamic linking

Bu usulda DLL funksiyalariga to‘g‘ridan-to‘g‘ri chaqiriqlar amalga oshiriladi. Buni faqatgina **header (.h)** va **import library (.lib)** fayllari orqali qilish mumkin. Quyida DLL’dan funksiyani chaqirish misoli:

```cpp
#include "stdafx.h"
#include "sampleDLL.h"
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
    HelloWorld();
    return 0;
}
```

### 2. Run-time dynamic linking

Bu usulda DLL alohida funksiyalar (**LoadLibrary** yoki **LoadLibraryEx**) yordamida ish vaqtida yuklanadi. Yuklangach, **GetProcAddress** orqali eksport qilingan funksiyani aniqlash kerak bo‘ladi. Misol:

```cpp
typedef VOID (*DLLPROC) (LPTSTR);
HINSTANCE hinstDLL;
DLLPROC HelloWorld;
BOOL fFreeDLL;

hinstDLL = LoadLibrary("sampleDLL.dll");
if (hinstDLL != NULL)
{
    HelloWorld = (DLLPROC) GetProcAddress(hinstDLL, "HelloWorld");
    if (HelloWorld != NULL)
        (HelloWorld);
    fFreeDLL = FreeLibrary(hinstDLL);
}
```

🔹 **Xavfli (malicious) kodlarda** tahdid aktorlari ko‘pincha **run-time dynamic linking** usulidan foydalanadilar. Chunki zararli dastur ba’zan fayllarni xotira hududlari o‘rtasida ko‘chirishi kerak bo‘ladi va bitta DLL’ni ko‘chirish boshqa fayl talablari orqali import qilishdan ko‘ra ancha oson boshqariladi.
