
suppose you found a weird service listening on port 5000, lets investigate
```
grep -R <PORT> /etc

cat /etc/nginx/sites-en/supper-pass-test.nginx
server_name test.superpass.htb
```

