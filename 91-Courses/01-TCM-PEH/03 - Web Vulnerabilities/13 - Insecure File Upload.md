# Lab 1
[PortSwigger Academy Insecure File Upload](https://portswigger.net/web-security/file-upload)
Open Burp  
Try file upload  

## File Extension Validation
If there is file extension checking, determine if it is client or server side
* open dev tools **ctrl+shift+c**
* Network tab
* reupload file, see if app calls server to check, if no, client-side
* If it is server side, we can do a couple of things
	* disable JS
	* intercept check in burp, and modify it
## Burp Repeater
Upload a valid file, and send the request to the repeater  
Delete all requests contents, add PHP payload, and replace filename extension to match the validation check. 

### Payload

```php


```

```

# Lab 2