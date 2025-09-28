# any cli tool with proxychains
To use `proxychains`, we first have to edit `/etc/proxychains.conf`, comment out the final line and add the following line at the end of it:


```shell-session
#socks4         127.0.0.1 9050
http 127.0.0.1 8080
```

```
$ proxychains curl http://SERVER_IP:PORT
```

# Nmap
```shell-session
]$ nmap --proxies http://127.0.0.1:8080 SERVER_IP -pPORT -Pn -sC
```
or using proxychains
```
 proxychains nmap SERVER_IP -pPORT -Pn -sC
```
# metasploit
```shell-session

msf6 auxiliary(scanner/http/robots_txt) > set PROXIES HTTP:127.0.0.1:8080
```