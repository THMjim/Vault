# AD-To-Do-Checklist

*Copy this to Box Directory to use as checklist*
---
## User Enumeration [](https://field-manual.brunorochamoura.com/manual/procedures/active-directory-compromise/#user-enumeration)
**With foothold**

- [ ]  Get user list via SMB

## Get Foothold [](https://field-manual.brunorochamoura.com/manual/procedures/active-directory-compromise/#get-foothold)

- [ ]  Find Kerberoastable users from the user list
- [ ]  Find ASREProastable users from the user list
- [ ]  SMB credentialed spider through SMB shares
- [ ]  Use Responder to catch credential hashes (-A analyze mode)
- [ ]  Get `LOCAL SYSTEM` on Domain connected host to get a Computer account
- [ ]  Try password spraying users.txt and passwords.txt to check for password reuse.

Password spraying can lock accounts due to repeated failed attempts and should be used cautiously.
## Attacks [](https://field-manual.brunorochamoura.com/manual/procedures/active-directory-compromise/#attacks)

- [ ]  Use SharpHound to collect data to feed BloodHound
- [ ]  Check compromised hosts on BloodHound for outbound attack paths
- [ ]  Use NetExec to check for command execution via SMB, WinRM, and RDP for each compromised user.
- [ ]  Kerberoast
- [ ]  ASREProast
- [ ]  Look for credentials in GPOs (`gpp_password`, `autologin`)
- [ ]  For each compromised user, spider through readable SMB shares for sensitive information
- [ ]  For each compromised user, conduct SMB Hash Theft attacks on writable SMB shares
- [ ]  Look for passwords in user’s description fields
- [ ]  Check the DC’s SYSVOL SMB share for scripts containing credentials
- [ ]  [NoPac](https://github.com/cube0x0/CVE-2021-1675)
- [ ]  [PrintNightmare](https://github.com/m8sec/CVE-2021-34527)
- [ ]  [PetitPotam](https://github.com/topotam/PetitPotam)
- [ ]  Try compromised local administrator hashes on other hosts
- [ ]  Look for users with the `PASSWD_NOTREQD` field
- [ ]  Password spray using previously found passwords
- [ ] If share write access, try EVIL LNK, with resonder/impacket-ntlmrelayx