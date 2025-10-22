---
title: 
tags: []
phase: 
tool: []
---
## WWW

```bash
feroxbuster -u http://192.168.IP:80/ -t 100 -x "txt,pdf,html,php,asp,aspx,jsp" -k -q -o "./scans/http_feroxbuster_dirbuster.txt"
curl -sSik http://192.168.IP:80/
gobuster dir -u http://IP/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt

nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN "./scans/tcp_80_banner_nmap.txt" 192.168.IP
nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://192.168.IP:80 2>&1 | tee "./scans/nikto_tcp_80.txt"
whatweb --no-errors -a 3 -v http://192.168.IP:80 
wpscan --url http://URL/  --enumerate vp,u,vt,tt --verbose -o ./scans/wpspan_target.log  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY
```
