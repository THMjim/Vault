# WWW Enumeration
```bash
# /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # FEROX Default: 
feroxbuster -u http:/192.168.205.10/ -t 100  -x "txt,html,php,asp,aspx,jsp" -v -k -n -q -e -r -o "./scans/ferox_80.txt"

feroxbuster -t 100 -x "txt,html,php,asp,aspx,jsp" -k -q -o "./scans/feroxbuster_recurse_80.txt" -u http:/192.168.205.10/ 
feroxbuster -t 100 -x "txt,html,php,asp,aspx,jsp" -k -q -o "./scans/feroxbuster_recurse_9090.txt" -u http:/192.168.205.10:9090/ 


ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt  -o ./scans/ffuf_dirs_80.html -of html -u http://192.168.205.10/FUZZ/ 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt -o ./scans/ffuf_files_80.html -of html -u http://192.168.205.10/FUZZ/ 

ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt  -o ./scans/ffuf_dirs_9090.html -of html -u http://192.168.205.10:9090/FUZZ/ 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt -o ./scans/ffuf_files_9090.html -of html -u http://192.168.205.10:9090/FUZZ/ 




whatweb --color=never --no-errors -a 3 -v http://192.168.205.10:9090 2>&1 

nikto -ask=no -Tuning=x4567890ac -nointeractive -host http://192.168.205.10:80 2>&1 | tee "./scans/nikto_80.txt"

curl -sSikf http:/192.168.205.10/robots.txt


wpscan --url http://192.168.205.10:80 --enumerate vp,u,vt,tt --verbose -o "./scans/wpscan_80.txt"  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY  
wpscan --url http://192.168.205.10:9090 --enumerate vp,u,vt,tt --verbose -o "./scans/wpscan_9090.txt"  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY  

############


whatweb --color=never --no-errors -a 3 -v http://blaze.offsec/ 2>&1

http://blaze.offsec/ 

ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt  -o ./scans/ffuf_dirs_blaze.html -of html -u http://blaze.offsec/FUZZ/ 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-files-lowercase.txt -o ./scans/ffuf_files_blaze.html -of html -u http://blaze.offsec/FUZZ/  

```