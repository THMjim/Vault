# zshrc 

```bash

# Jimbo Custom stuff

# prompt with TUN 0 
export PROMPT=$'%F{blue}┌──(%F{green}%n@%m%f%F{blue})-[%F{reset}%~%F{blue}] %F{red}$(ip -4 addr show tun0 2>/dev/null | awk "/inet / {split(\\$2,a,\\\"/\\\"); print a[1]}")%f\n%F{blue}└─%F{cyan}$%f '

export rockyou='/usr/share/wordlists/rockyou.txt'
export PATH=$PATH:/opt/ligolo-ng/

# VPN fix MTU
alias lmtu='cat /sys/class/net/tun0/mtu'
alias smtu='ip link show tun0'
alias showmtu='ip link show tun0'
alias setmtu='sudo ip link set tun0 mtu 1360'

```