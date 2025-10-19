# NOTES
**bold**  
*italic*  
***bold and italic***  
url link <google.com>  
email link <jimbo@gmail.com>  
Bold link This is the *[Markdown Guide](https://www.markdownguide.org)*  
Bold link This is the **[Markdown Guide](https://www.markdownguide.org)**  

## SMB  
### metasploit
msfconsole
search smb
`use auxiliary/scanner/smb/smb_version`  (or use number)
info  
set RHOSTS TARGET_IP  
run  
*update notes with samba version found*  

### smbclient
smbclient -l \\\\targetip\\  
smbclient -l \\\\targetip\\ADMIN$  

## SSH
`ssh IP -oKexAlgorithms=+diffie-hellman-group1-sha1 -c aes128-cbc`
you don't need cypher and algorithms, but its important to know syntax


***at this point, in our ntoes, we'll have some information about open servcies, version of services, vulnerabilities already found, such as information discolsure, default web page showing lax security***

review google for CVEs  
mod ssl 2.8.4 exploit, Apache httpd 1.3.20 exploit,samba 2.2.1a exploit
exploit-db, github, rapid7
Update notes : 80/443 - potentially vulnerable to OpenLuck, links to exploi-db, link to github  
Update notes : 139 - potentially vulnerable to trans2open
Update notes : save link to posible code exploits  
Rapid7 link will have MSF module options listed, making exploit/breach much easier  

### MSF searchsploit  
searchsploit Samba 2  
searchsploit mod ssl 2  

### notes so far
- ensure your screenshots of IPs visible  
- borders or highlights  around important information  
- machine name, with ip, `KIOPTRIX (192.168.57.134)`  
- nmap subfolder  
-- individual sub folders below nmap, per service, 22,80/443,139  
-- 80/443 has sub notes for different results, nikto/dirbuster, etc  
- Exploitation  
- Findings section  
-- Default web page - apache  
-- Information Disclosure  

## Nessus
- host scan
- review vulnerabilties
-- notes: insufficient patching  













