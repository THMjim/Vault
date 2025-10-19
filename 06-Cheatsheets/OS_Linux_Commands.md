# Linux OS Commands
## Loop Bash
* For starts loop, Done ends
```bash
for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips.txt     # done >  ips.txt outputs to file
```
## Count Words in File
```bash
wc -w /usr/share/wordlists/rockyou.txt   
```
##  Update Hosts File
```bash
echo -e "192.168.0.100\toffsecwp" | sudo tee -a /etc/hosts
# remove last line in file with SED
sudo sed -i '$d' /etc/hosts
```
## Add Words to Wordlist
```bash
echo -e "daniel\njustin" | sudo tee -a /usr/share/wordlists/dirb/others/names.txt
# \n  : carriage return, daniel is on one line, <\n enter>, justin is on the next line
```
## Hash
```bash
echo -n "secret" | sha256sum
# -n strips new  line
```
## TMUX 
- [ipsec](https://www.youtube.com/watch?v=Lqehvpe_djs)
- [network chuck](https://www.youtube.com/watch?v=nTqu6w2wc68)
- [hammond](https://www.youtube.com/watch?v=5-_bUD6oMok)

```bash
tmux  new  -s VPN  
tmux ls
tmux kill -server
tmux a -t VPN

# While in TMUX
# prefix key : ctrl + b  (Cb)
Cb + d     #  detach, go back to normal terminal

Cb + c     # new window
Cb + $     #  rename session
Cb + ,     #  rename window
Cb + n|p   #  navigate next previous window
Cb + 0|1|2 #  navigate to window # 

Cb + %        # vertical split
Cb + "     "  # horizontal split
Cb + z        # zoom in/out maximize current split you're in
Cb + spacebar # cycle through pre-set layouts 

Cb + w                # menu to move windows/sessions
Cb + U|D|L|R arrow    # move around  splits in terminal

Cb + Alt + Shift +  P # save to log file with plugin

##########################################
#  ~/.tmux.conf Set prefix to Ctrl-Space instead of Ctrl-b
unbind C-b
set -g prefix C-Space
bind C-Space send-prefix

set -g base-index 1          # Start session ordering at 1
setw -g pane-base-index 1    # Start window ordering at 1
###########################################
```
## Lab setup
```shell
# if you can't connect to web app, lower mtu 
sudo ifconfig tun0 mtu 1360
# OR
sudo ip link set tun0  mtu 1360

# we can add a ~/.bash_aliases to provide shortcusts
# even better, update .zshrc, and use it to build aliases for stuff like rockyou.txt
# aliases in .zshrc don't need to be re-sourced everytime terminals load
alias changeMyMTU='sudo ip link set tun0  mtu 1360'
alias showMyMTU='ip link show tun0'

# OR
cat /sys/class/net/tun0/mtu

# reload shell with SOURCE command
source ./.bash_aliases
```
## Update Kali
* Note, updating KALI broke my mouse cursor
* Take VM snapshot before attempting anything below
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove
sudo apt autoclean
```
