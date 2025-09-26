os,
kernel version,
processes
services
home (.ssh .history)
sudo 
passwd, shadow
ls (suid guid)

cronjobs
writeable dirs/files

`dirs`
```shell-session
$ find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```
`files`
```shell-session
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

 File Systems & Additional Drives
