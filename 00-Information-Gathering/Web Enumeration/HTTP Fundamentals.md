# HTTP Fundamentals

TCM - https://www.youtube.com/watch?v=qv5-5hhsKpE&list=PLLKT__MCUeixCoi2jtP2Jj8nZzM4MOzBL&index=10
Q: What is HTTP
A: HTTP is a protocol.
Q: What is a protocol
A: A protocol is a set of rules and guidelines that govern the communication between 2 or more entities

REQUEST / RESPONSE
REQUEST: 
Sent by client.
Request is made of of methods, URI (uniform resource Identifier), & message headers
Methods : GET PUT POST DELETE
URI: Uniform Resource Identifier, identifies resource being requested
Message Headers: Additional info about the request

RESPONSE
Sent by server.
status code
message headers
body

Requests are done with URLs.
https://tcm-sec.com/blog?topic=appsec#part3

## Status Codes
- 100_199 : Information responses
- 200-299 : Success messages
- 300-399 : Redirects
- 400-499 : Request Error
- 500-599 : Server Error

## Headers and Cookies
- HOST: what site your after, in case of Virtual HOST/VTLs
- User Agent: which browser you're running
- Content Length: used for form submittal or JSON, how much data to expect
- Accepted Encoding: what data we support
- Cookies: set with set cookie header, has multiple flags
- Cookies have a number of flags
	- http-only, restricts cookie being read by JS
	-  secure, only send cookie over TLS
	-  same-sites: strict /lax /none, what to do on cross site requests


## curl to submit da
```bash
curl --data '{"name":"alex"}' https://tcm-sec.com/
curl -X "POST" -d "mydata" https://tcm-sec.com/
```

