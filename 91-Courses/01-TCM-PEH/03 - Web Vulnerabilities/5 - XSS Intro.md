# XSS - Cross Site Scripting
## Intro  
3 main types of XSS   #xss
* Reflected
* Stored
* DOM-based
Anything we can do in Javascript/js, we can do in XSS
https://www.w3schools.com/js/  
* go through 20 minutes, maybe down to functions for a foundation of js*

### Reflected
Reflected  XSS is where the script you're trying to inject, comes from the current http request.  You send a request, and you get a response.  The script is included in the response. It's limited, as you can only target yourself, unless payload is via URI, and maybe you entice a user to click on link.
![[5 - XSS-reflected.png]]

### Stored
Payload is stored in something like a DB, and then retrieved later.  Allows you to attack other users.
![[5 - XSS-Stored.png]]


### DOM-Based
Client-side has some vulnerable js that uses untrusted input, instead of having a vuln server-side.  Everything happens locally in the browser.


### Example / Demo
Open browser and developer tools console, "Ctrl+shift+c"
```js
alert1() 
```
* generally speaking, don't user alert, as everyone detects it.
* When testing xss, use print or pompt instead.
```js
print()
prompt("hello")
```
### Logkey Demo
* demo of simple function to log keys with js/css
```js
function logKey(event){console.log(event.key)}
document.addEventListener('keydown',logKey)
```