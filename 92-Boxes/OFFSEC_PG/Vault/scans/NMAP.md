```bash
# ALL ports, search OPEN
sudo nmap -vv --reason -Pn -T4 -p- -oN "./scans/nmap_open_tcp.txt" 192.168.IP 

sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "./scans/nmap_tcp_detailed.txt"  -p 22,80 192.168.IP   

# top 2000 ports
sudo nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "./scans/nmap_quick_2000.txt"  --top-ports 2000 192.168.IP    

# UDP 
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/nmap_UDP_20.txt" <IP>  	
```









```