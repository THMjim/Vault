# Intro
* #commandInjection  
* Serious, because when we find it, we can often compromise the application and the host.
* Application is taking input from a user, and passing it to a function that executes it as code.  "Eval is Evil"
```js
eval(1+1)
2
let userInput = '7*7'
eval(userInput)
49
```

* NEVER trust user input!
```shell
└─$ php -a             
Interactive shell

php > $userInput = 'whoami';
php > system($userInput);
kali
```

* https://appsecexplained.gitbook.io/appsecexplained/common-vulns/command-injection
* 