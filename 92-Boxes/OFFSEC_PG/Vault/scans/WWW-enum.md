# WWW Enumeration

```bash
# /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # FEROX Default: 
# NON-recursive
feroxbuster -u http:/<IP>/ -t 100  -x "txt,html,php,asp,aspx,jsp" -v -k -n -q -e -r -o "./scans/ferox_80.txt"
# Recursive
feroxbuster -t 100 -x "txt,html,php,asp,aspx,jsp" -k -q -o "./scans/feroxbuster_recurse_80.txt" -u http://192.168.IP:80/ 
```

---

```bash
# directories and files, not recursive
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt  -o ./scans/ffuf_dirs_80.html -of html -u http://ip/FUZZ/ 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt -o ./scans/ffuf_files_80.html -of html-u http://IP/FUZZ

-o ./scans/ffuf_files_80.html -of html
-o ./scans/ffuf_dirs_80.html -of html

# feroxbuster probably better here, as uses medium word list, we can add extensions -x php,html,js,txt etc..
# ferox default depth is 4
```

---

```bash
whatweb --color=never --no-errors -a 3 -v http://192.168.110.201:80 2>&1 

nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://<IP>:80 2>&1 | tee "./scans/nikto_80.txt"

curl -sSikf http://192.168.110.201:80/robots.txt
curl -sSik http://192.168.110.201:80/
```
