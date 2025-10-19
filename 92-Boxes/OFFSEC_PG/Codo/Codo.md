---
box_name: Codo
author: James Heringer
platform: Offsec
date: 2025-10-12
tags: []
difficulty: easy
my_rating: easy
---
# Codo
- 192.168.193.23
## TLDR Summary 
1. Enumerate, identify webserver running CODO
2. Feroxbuster shows Codo has /admin/ page
3. Successful login to admin portal,  wtih default  creds admin : admin
4. *searchsploit codo*  gives us a listing for a codo forum, with multiple exploits
5. `searchsploit -x 50978` gives us a   path to upload a file  with RCE  `/sites/default/assets/img/attachments/`
6. In admin  panel, global config, *Upload logo for your forum*, we  are able to upload simple php rev shell
7. Activate RevSHell using our previously discovered path, of  `http://url/sites/default/assets/img/attachments/RevShell.php`
8. Have  shall on server, whoami shows  *www-data*
9. Copy over linpeas.sh and run, outputing to file
10. Linpeas output *passwords in* section shows config.php file with password
	- cat fastpeas.txt | grep -C 5 -i 'passwords in '
11. We use this password and su root,  to get flag
## Enum
```bash
sudo nmap -Pn -p- -T4 192.168.193.23 --reason 
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 61
80/tcp open  http    syn-ack ttl 61

sudo nmap  -p 22,80 -T4 192.168.193.23 -sC -sV -vv     

feroxbuster -u  http://192.168.193.23 -d 2 -k --silent 
```
- Browsing to the webpage, we find a register link
- Let's register! Send to burp
- Created new account *bob:password*
- Edit profile, there is a *change avatar* function, which looks for a file upload
- Changed avatar, with test.txt dummy text file and saved changes.
- Right-clicked avatar image, revealed its location - verified in curl
```bash
# verify file location
http://192.168.193.23/sites/default/assets/img/profiles/icons/68eb735214d67.txt
```
- attempted to upload  simple-php rev shell, but php extension not allowed
- Was able to upload as jpg,  but no execution
- Attempting client-side filter bypas
- We know we can upload .jpg, .txt, but when we upload our php code  as jpg, it doesn't execute.
- Burp Intercept ON, Taking post, and swapping content type
- Content-Type: image/jpeg  Content-Type: application/x-php
```bash
codo.php%00.png%00.jpg"

```

- http://192.168.193.23/admin/index.php
- logged in with admin:admin
- Under global settings, added php to the allowed upload types
- Added allowed Mimetypes : application/x-php
- Now we changed bobs Avatar again with  our  codo.php, browsed to it, and we  are on the box with a shell
##  PrivEsc
- transferrered linpeas.sh script, ouput
- 
```bash 
# sudo version https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-version
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
hydra -l offsec -P /usr/share/wordlists/rockyou.txt ssh://192.168.193.23  
# gave up after 5 min

# check for  world writable files
find /  -type f -perm 777  2>/dev/null


```

- In  www directory, there is a file named : */var/www/html/sites/default/config.php* 
```php
  $config = array (
  'driver' => 'mysql',
  'host' => 'localhost',
  'database' => 'codoforumdb',
  'username' => 'codo',
  'password' => 'FatPanda123',
  'prefix' => '',
  'charset' => 'utf8',
  'collation' => 'utf8_unicode_ci',
);

```
##  FLAG

- root : FatPanda123
- 232d65f9126bf6f3ea6c9cc1fdb7c214

