---
title: Information Gathering Quick Reference 
tags: [TTP, enum]
phase: enum
tool: [feroxbuster,ffuf]
---
# Information Gathering - Quick Reference
## Wordlists

```bash
/usr/share/wordlists/rockyou.txt             # 15 mil
/usr/share/wordlists/dirb/common.txt         # 4,500
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # 30,000 FEROX Default: 
/usr/share/wordlists/dirb/small.txt          # 900
/usr/share/seclists/Discovery/DNS/namelist.txt                      # BIG 150000
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt   # SMALL 5000
```
## NMAP 
```bash
# ALL ports, search OPEN
sudo nmap -vv --reason -Pn -T4 -p- -oN "./scans/nmap_open_tcp.txt" 192.168.IP 
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "./scans/nmap_tcp_detailed.txt"  -p 22,80 192.168.IP   
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "./scans/nmap_quick_2000.txt"  --top-ports 2000 192.168.IP    # top 2000 ports
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/nmap_UDP_20.txt" <IP>  	# UDP 
```

## WWW

```bash
feroxbuster -u http://192.168.IP:80/ -t 100 -x "txt,html,php,asp,aspx,jsp" -k -q -o "./scans/http_feroxbuster_dirbuster.txt"
curl -sSik http://192.168.IP:80/
gobuster dir -u http://IP/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt

nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN "./scans/tcp_80_banner_nmap.txt" 192.168.IP
nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://192.168.IP:80 2>&1 | tee "./scans/nikto_tcp_80.txt"
whatweb --no-errors -a 3 -v http://192.168.IP:80 
wpscan --url http://URL/  --enumerate vp,u,vt,tt --verbose -o ./scans/wpspan_target.log  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY
```
## DNS
```bash
dnsrecon -d megacorpone.com -t std   # check SOA, records, attempt Zone XFER
dnsrecon -d megacorpone.com -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt  -t brt  # BruteForce subdomains 

# Build hoss file from DC
nxc smb 192.168.205.172 -u "guest" -p "" --generate-hosts-file ./scans/hosts

```

