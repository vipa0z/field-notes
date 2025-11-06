1. check if web app does not rely on correct extension ( server dynamically adds ext )
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```
1. assuming there's an index page, find valid extensions
```shell-session
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ

.php
```
look for common pages, with php ext
```shell
## Page Fuzzing
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php

gobuster dir -u http://dnlanefight.local -w /usr/share/wordlists/dirb/common.txt -x .php -t 300
2022/06/20 22:04:48 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 288]
/.htpasswd            (Status: 403) [Size: 288]
/.hta                 (Status: 403) [Size: 288]
/.htpasswd.php        (Status: 403) [Size: 288]
/.hta.php             (Status: 403) [Size: 288]
/upload.php           (Status: 200) [Size: 14]                                               

```
