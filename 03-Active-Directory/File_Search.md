# File Search
File Names
```powershell
cd c:\Users
tree /F

Get-ChildItem -Path C:\ -Include flag.txt, proof.txt,local.txt -Recurse -ErrorAction SilentlyContinue
```

Text inside of  File
```powershell
gci *.md -Recurse -file  -ErrorAction SilentlyContinue  | Select-String -Pattern "curl.*%2e" # powershell

Get-ChildItem -Path C:\  *.txt -Recurse -file   -ErrorAction SilentlyContinue  | Select-String -Pattern "howdy" # powershell
```
## KDBX
1. In Windows

```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

2. In Linux

```bash
find / -name *.kdbx 2>/dev/null
```
