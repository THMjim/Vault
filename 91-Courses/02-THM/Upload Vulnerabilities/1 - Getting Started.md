---
type: notes
tags:
  - THM
  - www
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabiliites/Intro
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---

# Task 1 - Getting Started

Target IP = 10.10.86.124
Tun0 IP = 

## Configure Hosts File
This is need for DNS resolution, or when Virtual Hosting, commonly shortened to ***vhosting***,  is used to server multiple websites from a single server .  Also used when subdomains are mapped to single IP.  
## Update HOSTS file  for lab : 10.10.86.124 
#virtualhosts 
```shell
echo '10.10.86.124    overwrite.uploadvulns.thm shell.uploadvulns.thm java.uploadvulns.thm annex.uploadvulns.thm magic.uploadvulns.thm jewel.uploadvulns.thm demo.uploadvulns.thm' | sudo tee -a /etc/hosts

# Enumerating Virtual Hosts
ffuf -u http://<target-ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   -H "HOST: FUZZ.target-domain.com" -fs XX
```

# Task 2 - Intro 
The purpose of this room is to explore some of the vulnerabilities resulting from improper (or inadequate) handling of file uploads. Specifically, we will be looking at:

- Overwriting existing files on a server
- Uploading and Executing Shells on a server  
    
- Bypassing Client-Side filtering
- Bypassing various kinds of Server-Side filtering
- Fooling content type validation checks
# Task 3 - General Methodology

Looking at the source code for the page is good to see if any kind of client-side filtering is being applied. Scanning with a directory bruteforcer such as Gobuster is usually helpful in web attacks, and may reveal where files are being uploaded to; Gobuster is no longer installed by default on Kali, but can be installed with `sudo apt install gobuster`. Intercepting upload requests with [Burpsuite](https://tryhackme.com/room/burpsuitebasics) will also come in handy. Browser extensions such as [Wappalyser](https://www.wappalyzer.com/download) can provide valuable information at a glance about the site you're targetting.  

If the website is employing client-side filtering then we can easily look at the code for the filter and look to bypass it (more on this later!). If the website has server-side filtering in place then we may need to take a guess at what the filter is looking for, upload a file, then try something slightly different based on the error message if the upload fails. Uploading files designed to provoke errors can help with this. Tools like Burpsuite or OWASP Zap can be very helpful at this stage.