# WWW Enumeration
```bash
# /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # FEROX Default: 
feroxbuster -u http:/<IP>/ -t 100  -x "txt,html,php,asp,aspx,jsp" -v -k -n -q -e -r -o "./scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt"

whatweb --color=never --no-errors -a 3 -v http://192.168.110.201:80 2>&1 

nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://<IP>:80 2>&1 | tee "./scans/tcp80/tcp_80_http_nikto.txt"

curl -sSikf http://192.168.110.201:80/robots.txt
curl -sSik http://192.168.110.201:80/
```