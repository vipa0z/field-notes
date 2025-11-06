- prefer  ping scanning from compromised hosts (more reliable)
-  do ICMP/ARP first (faster, clean results).
- If it’s a **defensive / filtered environment** → avoid relying only on ICMP. Use `-PR` (ARP) or `-Pn`.
## HOST DISCOVERY
### nmap
perform host discovery, send ICMP echo requests
```bash
sudo nmap -sn -pE --disable-arp-ping 
 subnet/cidr -oA nmap/ecocorp | grep for | cut -d" " -f5 -oA nmap/targetname
```
use arp scanning:
```
nmap -PR -sn 192.168.1.0/24
```

`send icmp ` [Fping](https://fping.org/)   issue ICMP packets against a list of multiple hosts at once.
```shell
$ fping -asgq 172.16.5.0/23

172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.240

       9 alive
```

## oneliner scripts

`bash`
```shell-session
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

#### Ping Sweep For Loop Using CMD

```cmd-session
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

#### Ping Sweep Using PowerShell

show hostname + ip
```
1..254 | % {"172.16.210.$($_): $(Test-Connection -Count 1 -ComputerName 172.16.210.$($_) -Quiet)"}
```


```powershell-session
1..254 | % {"172.16.<9>.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

---
## PORT SCANNING A SUBNET
## rustscan 

```
rustscan -a 192.168.1.0/24 --ulimit 5000 -- -sV -sC  -oA nmap/eco-corp 
```


---

