# RLWRAP
* rlwrap is a wrapper program that adds the features  to programs that don't natively have them
* NCAT, upgraded NC version, supports reconnect, SSL
```bash
rlwrap ncat -nvlp <port>
rlwrap --prompt-colour=red --complete-filenames --ansi-colour-aware --history-no-dupes 2 --logfile <logfile> --remember ncat -nvlp <port>
```