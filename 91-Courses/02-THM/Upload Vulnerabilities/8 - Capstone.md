---
type: notes
tags:
  - THM
  - Filtering
  - WWW
  - js
  - javascript
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/Capstone
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---
# Capstone!
## Grab  HTTP Header 
```shell
└─$ curl -IL http://jewel.uploadvulns.thm                                       
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Sun, 01 Dec 2024 10:09:47 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 1514
Connection: keep-alive
X-Powered-By: Express
```
Wappalyzer shows : 
![[8 - Capstone-wappalyzer.png]]


Express, so PHP doesn't work, need a Node.JS script.
![[8 - Capstone-express.png]]
Acquired shell code from revshells.com script for  Node.JS.
## Enumerating the Directories
```shell
gobuster dir -u http://jewel.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
/content              (Status: 301) [Size: 181] [--> /content/]
/modules              (Status: 301) [Size: 181] [--> /modules/]
/admin                (Status: 200) [Size: 1238]
/assets               (Status: 301) [Size: 179] [--> /assets/]
```


OK, had to remove the BURP filter for not saving/intercepting JS, now we see a client-side filter, that does the following: 
* checks file extension, must be jpg/jpeg
* checks file size, must be < a certain size
* checks magic number, must be  "ÿØÿ"  

Website is looking for a JPEG. 
Copied the jpeg magic number from the wiki :
https://en.wikipedia.org/wiki/List_of_file_signatures
`FF D8 FF EE`
Inserted JPEG magic number at top of  Node.JS `1.jpg` reverse shell. 
File upload succeeded.
Enumerate for files on server
### Notes
Inserting the magic # in the front of the file, broke the file and caused this not to execute.
The key was to disable the client-side filtering on the browser, leaving the payload alone, and then also just renaming the file to .jpg, to get around the server-side filtering.

## FOUND the files
```shell
feroxbuster --url http://jewel.uploadvulns.thm/  -w vulns.txt -x jpg 

─────────────────────────────────────────────────
http://jewel.uploadvulns.thm/content/ABH.jpg                               http://jewel.uploadvulns.thm/content/LKQ.jpg                               http://jewel.uploadvulns.thm/content/SAD.jpg
http://jewel.uploadvulns.thm/content/UAD.jpg

# New
 http://jewel.uploadvulns.thm/content/WUJ.jpg
```
Navigating to the file produces errors: ![[8 - Capstone-file error.png]]

Reuploading payload with the NODE.js payload #2 from revshells.com.
Re-enumerating to find the new  file.
```shell
# I think this is it
http://jewel.uploadvulns.thm/content/JMU.jpg

```



![[8 - Capstone-FLAG.png]]


Help if needed.. https://www.jalblas.com/blog/tryhackme-upload-vulnerabilities-walkthrough/