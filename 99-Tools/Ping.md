# Ping

We can identify OS with ping: 
 `ping -c 1 192.168.50.10 ` ttl  ~ 64 nix ~ 128 Windows ~ 254 Cisco devices

```bash
#### taken from John Hammond - How To Pivot Through a Network with Chisel, https://www.youtube.com/watch?v=pbR_BNSOaMk&t=1878s
# linux 
for i in $(seq 254); do ping 1.1.1.${i} -c1 -W1 & done | grep from

# Windows 
1..254 | ForEach-Object -Parallel {
    if (Test-Connection -TargetName "192.168.1.$_" -Count 1 -Quiet -TimeoutSeconds 1) {
        Write-Host "Host 192.168.1.$_ is up." -ForegroundColor Green
    }
}
```




