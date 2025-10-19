# NMAP
```bash
sudo nmap -vv --reason -Pn -T4 -p- -oN 192.168.IP
sudo nmap -vv --reason -Pn -T4 -sV -sC  -p 22,80,443 -oN "./scans/_specific_tcp_nmap.txt" 192.168.IP

# UDP
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/_full_tcp_nmap.txt" <IP>  














```