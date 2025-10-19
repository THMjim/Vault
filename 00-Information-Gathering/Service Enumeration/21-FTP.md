# FTP enumeration

```powershell
ftp <IP>
#login if you have relevant creds or based on nmap scan find out whether this has an anonymous login or not, then login with Anonymous:password

put <file> #uploading file
get <file> #downloading file

#NSE
locate .nse | grep ftp
nmap -p21 --script=<name> <IP>

#bruteforce
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://IP -I
hydra -L users.txt -P passwords.txt <IP> ftp #'-L' for usernames list, '-l' for username and vice versa

# Check for vulnerabilities associated with the identified version.
```