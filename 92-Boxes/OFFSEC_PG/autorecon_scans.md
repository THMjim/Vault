# AutoRecon Scans
## Automatic
### NMAP
```bash
# quick
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "./scans/_quick_tcp_nmap.txt" 192.168.110.203
# full
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN "./scans/_full_tcp_nmap.txt" 192.168.110.203

# UDP
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/_full_tcp_nmap.txt" <IP>  

```
### WWW
```bash
feroxbuster -u http://192.168.110.201:80/ -t 100 -w /root/.local/share/AutoRecon/wordlists/dirbuster.txt -x "txt,html,php,asp,aspx,jsp" -v -k -n -q -e -r -o "./scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt"

curl -sSikf http://192.168.110.201:80/.well-known/security.txt
curl -sSikf http://192.168.110.201:80/robots.txt
curl -sSik http://192.168.110.201:80/

nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://192.168.110.201:80 2>&1 | tee "/home/kali/Documents/Boxes/Offsec/results/192.168.110.201/scans/tcp80/tcp_80_http_nikto.txt"

# banner grab
nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN "./scans/tcp80/tcp_80_http_nmap.txt" 192.168.110.201

whatweb --color=never --no-errors -a 3 -v http://192.168.110.201:80 2>&1
```
### SMB
```bash
enum4linux-ng -A -d -v 192.168.110.203 2>&1

smbclient -L //192.168.110.203 -N -I 192.168.110.203 2>&1

# Permissions check
smbmap -H 192.168.110.203 -P 139 2>&1
smbmap -H 192.168.110.203 -P 445 2>&1

# NULL session
nxc smb 192.168.50.179 --username '' --password ''
smbmap -u null -p "" -H 192.168.110.203 -P 139 2>&1
smbmap -u null -p "" -H 192.168.110.203 -P 445 2>&1

# Anonymous I think, or default 
smbmap -H 192.168.110.203 -P 139 -r 2>&1
smbmap -H 192.168.110.203 -P 445 -r 2>&1


smbmap -H 192.168.110.203 -P 139 -x "ipconfig /all" 2>&1
smbmap -H 192.168.110.203 -P 445 -x "ipconfig /all" 2>&1

smbmap -u null -p "" -H 192.168.110.203 -P 139 -x "ipconfig /all" 2>&1
smbmap -u null -p "" -H 192.168.110.203 -P 445 -x "ipconfig /all" 2>&1


```

### Windows 
```bash
impacket-getArch -target 192.168.110.203

# NETBIOS RPC 
impacket-rpcdump -port 135 192.168.110.203

nbtscan -rvh 192.168.110.203 2>&1

nmap -vv --reason -Pn -T4 -sV -p 135 --script="banner,msrpc-enum,rpc-grind,rpcinfo" -oN "./scans/tcp135/tcp_135_rpc_nmap.txt"  192.168.110.203

nmap -vv --reason -Pn -T4 -sV -p 139 --script="banner,(nbstat or smb* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "./scans/tcp139/tcp_139_smb_nmap.txt"  192.168.110.203

nmap -vv --reason -Pn -T4 -sV -p 445 --script="banner,(nbstat or smb* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "./scans/tcp445/tcp_445_smb_nmap.txt"  192.168.110.203

nmap -vv --reason -Pn -T4 -sV -p 3389 --script="banner,(rdp* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "./scans/tcp3389/tcp_3389_rdp_nmap.txt"  192.168.110.203
```


## Manual
### WWW
```bash
wpscan --url http://<IP> --no-update -e vp,vt,tt,cb,dbe,u,m --plugins-detection aggressive --plugins-version-detection aggressive -f cli-no-color 2>&1 | tee "./scans/tcp80/tcp_80_http_wpscan.txt"

# [-] Credential bruteforcing commands (don't run these without modifying them):
hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 80 -o "./scans/tcp80/tcp_80_http_auth_hydra.txt" http-get://<IP>/path/to/auth/area

medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 80 -O "./scans/tcp80/tcp_80_http_auth_medusa.txt" -M http -h <IP> -m DIR:/path/to/auth/area

hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 80 -o "/home/kali/Documents/Boxes/Offsec/results/192.168.110.23/scans/tcp80/tcp_80_http_form_hydra.txt" http-post-form://192.168.110.23/path/to/login.php:"username=^USER^&password=^PASS^":"invalid-login-message"

medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 80 -O "./scans/tcp80/tcp_80_http_form_medusa.txt" -M web-form -h 192.168.110.23 -m FORM:/path/to/login.php -m FORM-DATA:"post?username=&password=" -m DENY-SIGNAL:"invalid login message"

```
### SMB

```bash
# [*] msrpc on tcp/135	[-] RPC Client:
rpcclient -p 135 -U "" 192.168.110.203

# [*] netbios-ssn on tcp/139#	[-] Bruteforce SMB
crackmapexec smb 192.168.110.203 --port=139 -u "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -p "/usr/share/seclists/Passwords/darkweb2017-top100.txt"

# [*] microsoft-ds on tcp/445	[-] Bruteforce SMB
crackmapexec smb 192.168.110.203 --port=445 -u "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -p "/usr/share/seclists/Passwords/darkweb2017-top100.txt"

# [-] Lookup SIDs
impacket-lookupsid '[username]:[password]@192.168.110.203'

# [-] Nmap scans for SMB vulnerabilities that could potentially cause a DoS if scanned (according to Nmap). Be careful:
		nmap -vv --reason -Pn -T4 -sV -p 139 --script="smb-vuln-* and dos" --script-args="unsafe=1" -oN "./scans/tcp139/tcp_139_smb_vulnerabilities.txt"  192.168.110.203

# [-] Nmap scans for SMB vulnerabilities that could potentially cause a DoS if scanned (according to Nmap). Be careful:
		nmap -vv --reason -Pn -T4 -sV -p 445 --script="smb-vuln-* and dos" --script-args="unsafe=1" -oN "./scans/tcp445/tcp_445_smb_vulnerabilities.txt"  192.168.110.203
```

### RDP
```bash
# [*] ms-wbt-server on tcp/3389	[-] Bruteforce logins:
hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 3389 -o "./scans/tcp3389/tcp_3389_rdp_hydra.txt" rdp://192.168.110.203

medusa -U "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e ns -n 3389 -O "./scans/tcp3389/tcp_3389_rdp_medusa.txt" -M rdp -h 192.168.110.203
```