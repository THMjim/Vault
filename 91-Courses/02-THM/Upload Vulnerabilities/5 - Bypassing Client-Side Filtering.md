---
type: notes
tags:
  - THM
  - Filtering
  - WWW
  - js
  - javascript
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/Filtering
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---

# Task 7
There are four easy ways to bypass your average client-side file upload filter:

1. _Turn off Javascript in your browser_ -- this will work provided the site doesn't require Javascript in order to provide basic functionality. 
2. _Intercept and modify the incoming page._ Using Burpsuite, we can intercept the incoming web page and strip out the Javascript filter before it has a chance to run. 
3. _Intercept and modify the file upload_. Where the previous method works _before_ the webpage is loaded, this method allows the web page to load as normal, but intercepts the file upload after it's already passed (and been accepted by the filter).
4. _Send the file directly to the upload point._ Why use the webpage with the filter, when you can send the file directly using a tool like `curl`? Posting the data directly to the page which contains the code for handling the file upload is another effective method for completely bypassing a client side filter. 
	* the syntax for such a command would look something like this: 
```shell
curl -X POST -F "submit:<value>" -F "<file-parameter>:@<path-to-file>" site
```
* To use this method you would first aim to intercept a successful upload (using Burpsuite or the browser console) to see the parameters being used in the upload, which can then be slotted into the above command.
## Client-side bypass, MIME check
As always, we'll take a look at the source code. Here we see a basic Javascript function checking for the MIME type of uploaded files:

![[5 - Bypassing Client-Side Filtering-mime check.png]]


In this instance we can see that the filter is using a _whitelist_ to exclude any MIME type that isn't image/jpeg.    


Our next step is to attempt a file upload. As expected, if we choose a JPEG, the function accepts it. Anything else and the upload is rejected.  
### Intercept Client Requests 

 * Start Burpsuite, and reload the page.   #js #intercept
	 * NOTE: when reloading, use ***Ctrl+F5***, because the JS will get cached, and you won't see it in intercept if it is already cached
 * We will see our own request to the site, but what we really want to see is the server's _response_, so right click on the intercepted data, scroll down to "Do Intercept", then select "Response to this request":
 ![[5 - Bypassing Client-Side Filtering-mime check intercept.png]]
 
* When we click the "Forward" button at the top of the window, we will then see the server's response to our request. Here we can delete, comment out, or otherwise break the js function before it has a chance to load.  
 * If we delete the function, we once again click Forward until the site has finished loading, and are now free to upload any kind of file to the website.
 

###  Burp Quirk
 It's worth noting here that Burp will not, by default, intercept any external Javascript files that the web page is loading. If you need to edit a script which is not inside the main page being loaded, you'll need to go to the "Options" tab at the top of the Burpsuite window, then under the "Intercept Client Requests" section, edit the condition of the first line to remove `^js$|`: 
![[5 - Bypassing Client-Side Filtering-burp-quirk.png]]

## Modify MIME contents
Same intercept as above, but instead  uploading a file with a legitimate extension and MIME type, then intercepting and correcting the upload with Burpsuite.
I uploaded a php shell, that had a PNG extension. This bypassed the client-check, but fails to run.  We are going to change the MIME type of the php shell  to _text/x-php_, and the extension from _png_ to _php_
![[5 - Bypassing Client-Side Filtering-mime modify.png]]
![[5 - Bypassing Client-Side Filtering-chagned.png]]

Hit forward to complete the file upload.

## Challenge
Navigate to http://java.uploadvulns.thm  
We are presented with file upload options that only accept PNG files.  
Viewing source code, we see JS file doing client-side checks.
If we disable JS completely, the page breaks.  
With the page loaded, doing the following: 
### STEPS
Enumerate the JAVA website with gobuster as before, to discover the images directory.
Delete the JS  filter as noted above in the  "Intercept Client Requests" section  
Open Burp, _Enable_ intercept  
Switch to Burp tab PROXY -> Intercept   
Refresh the page  , you should see HTTP GET request  
Right click this, -> Do Intercept -> Response to this request  
![[5 - Bypassing Client-Side Filtering-response.png]]
Hit Forward, and you will now see the response with the JS scripts embedded.
Find the JS script named "client-side-filter.js"
Delete this code block
Having deleted the function, we once again click "Forward" until the site has finished loading, and are now free to upload any kind of file to the website:
Make sure the FORWARD button has gone gray, and then you know you're done forwarding, and can continue the exploit.  

Now select the revshell file for upload, click upload, 
and navigate in the web browser to the revshell.


## Walk through for  Question 1

What is the flag in /var/www/?
THM{NDllZDQxNjJjOTE0YWNhZGY3YjljNmE2}

 
