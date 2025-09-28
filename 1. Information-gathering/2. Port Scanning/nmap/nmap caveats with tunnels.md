
nmap by default sends `SYN` scans which dont work in proxies
so you should use `-sT`  and  `-Pn` to disable pings as they dont work in tunnels
