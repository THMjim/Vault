---
title: "Chapter 06: Information Gathering"
author: "James Heringer"
course: "PEN-200"
date: 2025-10-10
tags: [DNS,DNSenum]
description: "Information Gathering"
draft: false
layout: Course Notes
---
# Information Gathering
## 1. Passive Reconnaissance 
### Whois
* can do forward and reverse lookups
* `whois megacorpone.com -h 192.168.50.251`
* `whois 38.100.193.70 -h 192.168.50.251`

### Google Hacking
* [Exploit DB Google Dorks](https://www.exploit-db.com/google-hacking-database)
* [DorkSearch](https://dorksearch.com/)
* `site:megacorpone.com filetype:txt`
* Title has words _index of_ AND _parent directory_
* `intitle:"index of" "parent directory" `

### Open Source
* GitHub or other repos can be good sources of intel.  Must login first.
* GitHub search `owner:megacorpone path:users`

### Shodan
* [Shodan](https://www.shodan.io/)
* Catalogues internet connected devices
* Must **register** account to use

### Security Headers
* Two sites we can use for PASSIVE analysis of SSL security posture of website
* [Security Headers](https://securityheaders.com/)
* [SSL Labs](https://www.ssllabs.com/ssltest/)

## 2. Active Reconnaissance
### DNS Enumeration
* Active information gathering via querying DNS records of target
* Enumerate with native **Linux** commands
```bash
host megacorpone.com
host -t mx megacorpone.com
host -t txt megacorpone.com
# Build a subdomain list, 
for ip in $(cat list.txt); do host $ip.megacorpone.com; done
```
####  DNS Auto Enumeration Tools
```
dnsenum megacorpone.com
dnsrecon -d megacorpone.com -t std
dnsrecon -d megacorpone.com -D ~/list.txt -t brt # BRT is for brute force
```
#### Windows DNS Enumeration
```shell
xfreerdp3 /u:student /p:lab /v:192.168.50.152
nslookup mail.megacorptwo.com
nslookup -type=TXT info.megacorptwo.com 192.168.50.151
```
### TCP/UDP Port Scanning Theory
* Simplest scan is with TCP Connect, A.K.A 3-way Handshake.
* Nmap -sT is TCP connect.
* You can simualte this with nc
```bash
nc -nvv -w 1 -z 192.168.50.152 3388-3390
# -w1 : wait 1 second for timeout
# -z  : Zero I/O .. scan
# 3399-3400 port range

nc -nv -u -z -w 1 192.168.50.149 120-123  # This is UDP
```

### Nmap Port Scanning
* Default Nmap scan  as ROOT is -sS, Syn *Stealh* Scan
* Syn Stealth  doesn't complete 3-way handshake, so it's faster. Uses RAW sockets.
* Default non-ROOT scan is -sT, TCP Connect, slower because of 3-way handshake. Also uses Berkley Socket API  instead of  RAW sockets.
* ***IMPORTANT*** : When scanning through a **Proxy** you MAY need to use -sT, TCP connect, so keep that in mind
```bash
sudo nmap -sU <IP> # UDP scan
nmap -sn 192.168.0.1-254 # network sweep

# Network sweep for live hosts, then output those  with grep | cut
nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt
grep Up ping-sweep.txt | cut -d " " -f 2

# only top 20 ports, 
# 3rd column in this file is port frequency, for top X ports: /usr/share/nmap/nmap-services
nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt

# grab Website headers
nmap --script http-headers 192.168.50.6

```
### Windows Scanning
```powershell
Test-NetConnection -Port 445 192.168.50.151
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null 
```

## SMB Enumeration
* SMB is port 445, NetBIOS is 139
* They are both required, so if you see one the other is there
```bash
ls -1 /usr/share/nmap/scripts/smb* # Nmap NSE SMB scripts

nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254
nmap -v -p 139,445 --script smb-os-discovery 192.168.50.152

sudo nbtscan -r 192.168.50.0/24 # UDP port 137, scans for NetBIOS names

nxc smb 192.168.194.0/24   # SMB network sweep with NetExec

net view \\dc01 /all # from Windows 

enum4linux -A <target_IP_address>
enum4linux -u <username> -p <password> -a <target_IP_address>
enum4linux -r <ip> # enumerate USERS  
```

## SMPT Enumeration
```shell 
nc -nv <IP> 25 # Version Detection
smtp-user-enum -M VRFY -U username.txt -t <IP> # -M means mode; it can be RCPT, VRFY, EXPN

# Sending email with valid credentials, the below is an example of Phishing mail attack
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.50.242 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```

##  SNMP Enumeration
```bash
# Nmap UDP scan, SNMP is on port 161
sudo nmap <IP> -A -T4 -p- -sU -v -oN nmap-udpscan.txt

snmpwalk -c public -v1 -t 10 <IP> #Displays entire MIB tree, MIB Means Management Information Base
snmpwalk -c public -v1 <IP> 1.3.6.1.4.1.77.1.2.25 #Windows User enumeration
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.4.2.1.2 #Windows Processes enumeration
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.6.3.1.2 #Installed software enumeraion
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.6.13.1.3 #Opened TCP Ports

#Windows MIB values
1.3.6.1.2.1.25.1.6.0 - System Processes
1.3.6.1.2.1.25.4.2.1.2 - Running Programs
1.3.6.1.2.1.25.4.2.1.4 - Processes Path
1.3.6.1.2.1.25.2.3.1.4 - Storage Units
1.3.6.1.2.1.25.6.3.1.2 - Software Name
1.3.6.1.4.1.77.1.2.25 - User Accounts
1.3.6.1.2.1.6.13.1.3 - TCP Local Ports
```



