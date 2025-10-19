 ### Additonal resources 
The Bug Hunter's Methodology - https://www.youtube.com/watch?v=uKWu6yhnhbQ
Nahamsec Recon Playlist - https://www.youtube.com/watch?v=MIujSpuDtFY&list=PLKAaMVNxvLmAkqBkzFaOxqs3L66z2n8LA

### Installing Go  
Pimp my KALI will install Go/Golang  

### Finding Subdomains with Assetfinder  
tomnomnom assetfinder <https://github.com/tomnomnom/assetfinder>    #subdomains
`go get -u github.com/tomnomnom/assetfinder`  
#### Usage 
`assetfinder tesla.com  >> tesla-subs.txt`  
`assetfinder --subs-only tesla.com  >> tesla-subs.txt`  
better to 
`assetfinder tesla.com  | 


#### Bash script to pull out subdomains pretty, with DIR creation
```
#!/bin/bash  

url=$1

if [ ! -d "$url" ]; then
        mkdir $url
fi

if [ ! -d "url/recon"]; then
        mkdir $url/recon  
fi

echo "[+] Harvesting subdomains with assetfinder..."
assetfinder $url >> $url/recon/assets.txt
cat $url/recon/assets.txt | grep $1 >> $url/recon/final.txt  
rm $url/recon/assets.txt

#echo "[+] Harvesting subdomains with Amass..."
#amass enum -d $url >> $url/recon/f.txt
#sort -u $url/recon/f.txt >> $url/recon/final.txt
#rm $url/recon/f.txt

echo "[+] Probine for alive domains..."
cat tesla.com/recon/final.txt | httprobe -s -p https:443 | sed 's/https\?:\/\///' tr -d ':443' >> $url/recon/alive.txt  

```
after script is built if it is named run.sh, execute with `./run.sh tesla.com`    
***  
### Finding Subdomains with Amass  (OWASP)
<https://github.com/owasp-amass/amass>   #subdomains

KALI `apt-get update  apt-get install amass`  

GO  
1. Install Go and setup your Go workspace
2. Download OWASP Amass by running `go install -v github.com/owasp-amass/amass/v4/...@master`  
3. At this point, the binary should be in $GOPATH/bin  

USAGE : `amass enum -d tesla.com`  
***  
### httprobe 
<https://github.com/tomnomnom/httprobe>  
INSTALL `go install github.com/tomnomnom/httprobe@latest`  

#### Identify if domains are ALIVE!  
`cat tesla.com/recon/final.txt | httprobe -s -p https:443 | sed 's/https\?:\/\///' tr -d ':443' `  
The sed command strips the https://, and the tr strips the port from output  

### gowitness / Screenshot Websites  
<https://github.com/sensepost/gowitness>   

PREP STEP, install dependcy that is broken.. but might not be needed if fixed now?  
`go get -u gorm.io/gorm`

`go install github.com/sensepost/gowitness@latest`  

`gowitness scan single --url "https://sensepost.com" --write-db`

`gowitness single "https://sensepost.com"`
***  

## AUTOMATION

sumrecon: https://github.com/thatonetester/sumrecon

TCM's modified script - https://pastebin.com/MhE6zXVt

sumrecon DEPENDENCIES
assetfinder - https://github.com/tomnomnom/assetfinder  
amass - https://github.com/OWASP/Amass  
`certspotter - #curl -s https://certspotter.com/api/v0/certs\?domain\=$url | jq '.[].dns_names[]' | sed 's/"//g' | sed 's/*.//g' | sort -u (set as alias)` 
sublist3r - https://github.com/aboul3la/Sublist3r  
httprobe - https://github.com/tomnomnom/httprobe  
waybackurls - https://github.com/tomnomnom/waybackurls  
whatweb - https://github.com/urbanadventurer/WhatWeb  
nmap - https://nmap.org/download.html  
eyewitness - https://github.com/FortyNorthSecurity/EyeWitness  







