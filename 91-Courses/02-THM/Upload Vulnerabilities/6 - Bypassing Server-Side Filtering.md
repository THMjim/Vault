---
type: notes
tags:
  - THM
  - Filtering
  - WWW
  - js
  - javascript
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/Filtering
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---
# Task 8
If serverside is filtering on PHP extension, we can try to use an alternate one from this list : <https://book.hacktricks.xyz/pentesting-web/file-upload>

- If they apply, the **check** the **previous extensions.** Also test them using some **uppercase letters**: _pHp, .pHP5, .PhAr ..._
    
- _Check_ _**adding a valid extension before**_ _the execution extension (use previous extensions also):_
 * file.png.php, file.png.Php5

 * Try adding **special characters at the end.** You could use Burp to **bruteforce** all the **ascii** and **Unicode** characters. (_Note that you can also try to use the_ _**previously**_ _motioned_ _**extensions**_)
	- _file.php%20_
	- _file.php%0a_
	- _file.php%00_
	- _file.php%0d%0a_
	- _file.php/_
	- _file.php.\_
	- _file._
	- _file.php...._
	- _file.pHp5...._        

- Try to bypass the protections **tricking the extension parser** of the server-side with techniques like **doubling** the **extension** or **adding junk** data (**null** bytes) between extensions. _You can also use the_ _**previous extensions**_ _to prepare a better payload._
	 - _file.png.php_
	- _file.png.pHp5_
	- _file.php#.png_
	- _file.php%00.png_
	- _file.php\x00.png_
	- _file.php%0a.png_
	- _file.php%0d%0a.png_
	- _file.phpJunk123png_      
 
* Add **another layer of extensions** to the previous check:
	- _file.png.jpg.php_
     - _file.php%00.png%00.jpg_

- Try to put the **exec extension before the valid extension** and pray so the server is misconfigured. (useful to exploit Apache misconfigurations where anything with extension** _**.php**_**, but** not necessarily ending in .php** will execute code):
    - _ex: file.php.png_
- Using **NTFS alternate data stream (ADS)** in **Windows**. In this case, a colon character “:” will be inserted after a forbidden extension and before a permitted one. As a result, an **empty file with the forbidden extension** will be created on the server (e.g. “file.asax:.jpg”). This file might be edited later using other techniques such as using its short filename. The  `"**::$data**"`   pattern can also be used to create non-empty files. Therefore, adding a dot character after this pattern might also be useful to bypass further restrictions (.e.g. “file.asp::$data.”)
    
- Try to break the filename limits. The valid extension gets cut off. And the malicious PHP gets left. AAA<--SNIP-->AAA.php

## Challenge
I uploaded a file, **revshell.jpg**, but I didn't find it on the server.
gobuster found the directory, 
```shell
gobuster dir -u http://annex.uploadvulns.thm/assets/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```shell
.php3
.php4
.php5
.php7
.phps 
.php-s 
.pht

```

I started uploading the above files, and finally my file with the .php5 succesfully uploaded.
After that I was able to navigate to it in the privacy directory, it had been given a random name at this point, and execute it to get a reverse shell.

FLAG THM{MGEyYzJiYmI3ODIyM2FlNTNkNjZjYjFl}
