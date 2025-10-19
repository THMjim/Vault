---
type: notes
tags:
  - THM
  - RCE
  - WWW
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/RCE
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---
# Task 5

There are two basic ways to achieve RCE on a webserver when exploiting a file upload vulnerability: webshells, and reverse/bind shells. Realistically a fully featured reverse/bind shell is the ideal goal for an attacker; however, a webshell may be the only option available (for example, if a file length limit has been imposed on uploads, or if firewall rules prevent any network-based shells)

## Web shells walkthrough
Let's assume that we've found a webpage with an upload form: 
![[3 - Remote Code Execution-upload form.png]]
### Step 1 - Enumerate Directories
We want to know the website directory structure to know where to look for our file that we upload. 
```shell 
# /usr/share/wordlists/dirb/common.txt 
feroxbuster --url http://demo.uploadvulns.thm --wordlist=/usr/share/wordlists/dirb/common.txt --silent -C 404 -d 1 
# OR
gobuster dir -u http://demo.uploadvulns.thm  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
```
![[3 - Remote Code Execution-dir scan.png]]
Here we see two directories
* /uploads  &  /assets 
It's most likely our file upload will be found in /uploads.
## CMD Shell
If we can upload images, we should be able to upload a PHP CMD shell.
![[3 - Remote Code Execution-php shell example.png]]
Once the shell is uploaded, we navigate to it and issue commands : 
![[3 - Remote Code Execution-example shell command.png]]

## Reverse Shell
Instead of using CMD shell in the browser, we can upload a reverse shell.  
1. Start NC listener 
```
nc -nvlp 1234
```
2. Upload shell file, 
* [pentest moneky](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) 
* /usr/share/webshells/laudanum/php/php-reverse-shell.php 
3. Navigate to file on webserver, thus activating reverse shell
	* The website should hang and not load properly -- however, if we switch back to our terminal, we have a hit!

## Task Questions 1
Q: What directory looks like it might be used for uploads?
A: /resources
## Task Questions 2
Q: Get either a web shell or a reverse shell on the machine.  
What's the flag in the /var/www/ directory of the server?  
A: THM{YWFhY2U3ZGI4N2QxNmQzZjk0YjgzZDZk}