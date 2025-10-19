# SETUP
* #xss/stored #js #javascript 
* http://localhost/labs/x0x02.php
* We need to seperate browsers to test all the XSS for this lab
* Install firefox containers, to allow for multiaccounts
* https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/
* Click on firefox containers ICON
	* Manage Containers
	* setup 2 containers, 1 /2
* open lab 2 on both containers

## Payload
### Test for HTML Injection
* when testing for XSS, start by testing for HTML injection
```js
<h1>test</h1>
```
* if this works, we do have HTML, injection
* if BOTH firefox containers, can see the HTML injection we will have Stored XSS
### Test for Stored XSS Vuln
```js
<script>prompt(1)</script>
```
* Every user that come to this page will be impacted by our payload, and see the prompt 1 box.
### Steal Cookie Example
```js
<script>alert(document.cookie)</script>
```

### Prevent Cookie Stealing
* BEST PRACTICE : set cookies to http only, which is a flag that will prevent javascript from accessing your cookie
