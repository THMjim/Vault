# NMAP
## 
```bash
sudo nmap -vv --reason -Pn -T4 -p- -oN 192.168.IP
sudo nmap -vv --reason -Pn -T4 -sV -sC  -p 22,80,443 -oN "./scans/_specific_tcp_nmap.txt" 192.168.IP
# UDP
sudo nmap -vv --reason -Pn -T4 -sU --top-ports 20 -oN "./scans/_full_tcp_nmap.txt" <IP>  
#############################################################
PORT     STATE SERVICE       REASON          VERSION
22/tcp   open  ssh           syn-ack ttl 125 OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 e0:3a:63:4a:07:83:4d:0b:6f:4e:8a:4d:79:3d:6e:4c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDHi6qqQGQteAgLLakGf2MPORvtZSeF1gAL03sfUo/E/jsObuAOjzHPfw1OIfAyctU/gv17d0sz17kGDXbg4In4vxlnavXxnB/L2ipQLhAkFLo1YZLLwyUs0j1jiSW65Ax12A20nH0F3hbuqKsuWIPywR9XXc0cwhyY3ET06ZCbVVvP8ChGMd97uZTDeZtrnskYMXmqnfQpOGPL6e022+K5ud3MboyYXTlD0rGyVHyLCCXeg9QdxsVR5mZ8hSVCrwdX8Q4tb4kLYi+wpPx1KCirHEk/8O8I7YuA5KR4AeaMVSlqUhlm9MXaNw+5WZD2xOue0ZPCx2gxgKgndoUDfAagZrHr3dzgMUDgUnKTbtL/EIZdyzKh5C1TrN+yplSpqYjK975Q1qas7SUmzanN9S/PnhFFer/0j6hTtGlRalbxX12i2ifC0314ifnHY3Zz+2VD49RARArwvZR+7KcGKP+5tpCl9IyVf1xSKFFnJ0cQdhuAgJnkCfQXaQLHTzqSbys=
|   256 3f:16:ca:33:25:fd:a2:e6:bb:f6:b0:04:32:21:21:0b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOpOTM37eUD8SUKAqFVSNV8ECDl5yUqV7a41c39cXyPE6EMeNbKxWvQDoTw+WEYArHFAEj3SZbFupIZoZmvjFuA=
|   256 fe:b0:7a:14:bf:77:84:9a:b3:26:59:8d:ff:7e:92:84 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICyZUVdHOqTpEFyvELlmc7sCN1XFXQo7RdZdTRTPO2uJ
80/tcp   open  http          syn-ack ttl 125 Apache httpd 2.4.51 ((Win64) PHP/7.4.26)
|_http-server-header: Apache/2.4.51 (Win64) PHP/7.4.26
|_http-generator: Nicepage 4.8.2, nicepage.com
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-title: Home
81/tcp   open  http          syn-ack ttl 125 Apache httpd 2.4.51 ((Win64) PHP/7.4.26)
|_http-server-header: Apache/2.4.51 (Win64) PHP/7.4.26
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Attendance and Payroll System
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
135/tcp  open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds? syn-ack ttl 125
3306/tcp open  mysql         syn-ack ttl 125 MySQL (unauthorized)
5985/tcp open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```