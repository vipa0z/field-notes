This section focuses on  Firewall and IPS Detection & Evasion

# Firewalls
------------
### Detecting Firewalls

- Closed port: RST flag response
- Filtered port: Challenging to determine if closed or not
- Long delay in response or fast rejection with error code 3 indicates firewall

### Firewall Evasion Techniques
scan by a different source ip
`-S Scans the target by using different source IP address.`
```shell-session
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0pin
```
`--dns-server` specify dns server (more trusted)
- Connect Scan: `-sT`
- ACK Scan: `-sA`
- SOURCE PORT 53
- Decoy Scanning: `-D RND:5`
- Fragmentation: `-f` or `--mtu`
### 2. Source Port Manipulation ( — source-port)

Firewalls typically block suspicious traffic based on source ports. However, some firewalls trust traffic originating from certain well-known ports, such as DNS (port 53) or HTTP (port 80). By specifying a trusted source port, you can trick the firewall into letting your scan through.

Source Port Manipulation
```
nmap --source-port 53 <target IP>
```

decoy scanning:
```
nmap -D RND:10 <target IP>
```

idle scanning:
```
nmap -sI <zombie IP> <target IP>
```

mac address spoofing:
```
nmap --spoof-mac <mac address> <target IP>
```
udp scanning (-sV flag is important as it sends dozens of request to the service hoping for a reply):
```
nmap -sV -sV $Ip
```
ipv6 scanning:
```
-6
```
ftp bouce attack scan
```
nmap -p 22,25,135 -Pn -v -b XXX.YY.111.2 ip
```
other scans:
`-sN` (NULL scan), `-sX` (Xmas scan).

fin scan works in linux (windows returns RST if closed or open so its useless)
```
sF
```
##### ACK scan (`-sA`)

This technique is useful for differentiating between stateful and stateless firewalls. As only packets with the ACK flag are sent, stateless firewalls can let them through, whereas stateful firewalls will check whether there is an associated connection.

##### Fragmentation

This technique allows packets to be sent in smaller fragments. This can bypass security equipment that does not handle fragmented packets correctly.

#### IP protocol scanning

This is a special type of scan that is not included in conventional port scanning techniques. The purpose of the IP protocol scan (`-sO`) is to identify the IP protocols supported by the target: ICMP, TCP, IGMP, etc.

Rather than scanning ports as in traditional scans, this method varies the value of the Protocol field (8 bits) in the IP header, making it possible to determine which protocols are accepted or filtered by the target equipment.
### 6. MAC Address Spoofing ( — spoof-mac)

Some networks grant access to certain MAC addresses or have whitelisted specific devices. Spoofing the MAC address of a trusted device can allow you to bypass the firewall and access the target network.

**Command:**
```
nmap --spoof-mac <mac address> <target IP>
```

## NSE bypass/detection scripts

| Script               | Type     | Technique                                | Simple Description                        |
|----------------------|----------|------------------------------------------|--------------------------------------------|
| `firewall-bypass`    | Bypass   | TCP fragmentation                        | Sneak through chopped packets              |
| `ip-id`              | Detection| IPID pattern                             | See if the real host replies directly      |
| `ipidseq`            | Detection| IPID sequence                            | Check if packet numbers are predictable    |
| `traceroute`         | Detection| TTL path tracing                         | See who blocks you along the way           |
| `sniffer-detect`     | Detection| Promiscuous mode baiting                 | Detect network sniffers                    |
| `http-methods`       | Detection| HTTP verb probing                        | Spot HTTP filtering                        |
| `ftp-bounce`         | Bypass   | FTP as proxy                             | Leverage internal scan via FTP             |
| `bypass-firewalls`   | Bypass   | Packet evasion                           | Confuse the firewall                       |
| `packet-tracer`      | Detection| Packet fingerprinting                    | Watch how the target handles odd packets   |



### UDP Port Scans
- Some admins forget to filter UDP ports
- use -`sU`  and combine with source port manipulation

### Source Port Manipulation
```bash
--source-port 53
```


1. Fragment packets (-f or --mtu):
   ```
   nmap -f  --mtu 8 <target>
   ```

2. Use decoys (-D):
   ```
   nmap -D RND:10 <target>
   ```

3. Spoof source IP address (-S):
   ```
   nmap -S <spoofed_ip> <target>
   ```

4. Use a specific source port (--source-port):
Sometimes if services run on the same port you are scanning, It may bypass firewall.
   ```
   nmap --source-port 53 <target>
   ```

5. Slow down the scan (-T<0-5>):
   ```
   nmap -T2 <target>
   ```

6. Use TCP connect scan (-sT):
   ```
   nmap -sT <target>
   ```

7. Perform UDP scans (-sU):
   ```
   nmap -sU <target>
   ```

8. Use uncommon TCP flag combinations:
   ```
   nmap -sN <target>  # NULL scan
   nmap -sF <target>  # FIN scan
   nmap -sX <target>  # Xmas scan
   ```

9. Idle scan (-sI):
   ```
   nmap -sI <zombie_host> <target>
   ```

10. Use scripts to manipulate packets:
    Various NSE scripts can be used for this purpose.

## 10. Example Advanced Scan
```bash
sudo nmap 10.129.128.167 -Pn --disable-arp-ping -n -D RND:10 -sV -p- --source-port 53 -vv --dns-servers 10.129.128.167
```
#  Intrusion Prevention Systems (IPs)
-----------------------
- Use Virtual Private servers, useful incase you get IP banned 
- Fragment packets: `-f` or `--mtu`
	Most IPs dont know how to scan fragmented packets
- Decoy scanning: `-D RND:5`
	if you specify multiple fake IPs and slide your IP within them it may bypass detection.
- Specify interface: `-e <interface>` 
	specify Ips to scan from 
- DNS proxying: `--dns-server <ns>,<ns>`
	Scan target using a DNS server, abuses trust between them.
5. Slow down the scan (-T<0-5>):

##  Example 
This scan combines multiple evasion techniques for a stealthy, comprehensive scan.
```bash
sudo nmap 10.129.128.167 -Pn --disable-arp-ping -n -D RND:10 -sV -p- --source-port 53 -vv --dns-servers 10.129.128.167
```

### Detecting Firewalls

- Closed port: RST flag response
- Filtered port: Challenging to determine if closed or not
- Long delay in response or fast rejection with error code 3 indicates firewall

### Firewall Evasion Techniques
- Connect Scan: `-sT`
- ACK Scan: `-sA`
- Fragmentation: `-f` or `--mtu`
- Decoy Scanning: `-D RND:5`

### UDP Port Scans
- Some admins forget to filter UDP ports
- use -`sU`  and combine with source port manipulation

### Source Port Manipulation
```bash
--source-port 53
```


1. Fragment packets (-f or --mtu):
   ```
   nmap -f  --mtu 8 <target>
   ```

2. Use decoys (-D):
   ```
   nmap -D RND:10 <target>
   ```

3. Spoof source IP address (-S):
   ```
   nmap -S <spoofed_ip> <target>
   ```

4. Use a specific source port (--source-port):
Sometimes if services run on the same port you are scanning, It may bypass firewall.
   ```
   nmap --source-port 53 <target>
   ```

5. Slow down the scan (-T<0-5>):
   ```
   nmap -T2 <target>
   ```

6. Use TCP connect scan (-sT):
   ```
   nmap -sT <target>
   ```

7. Perform UDP scans (-sU):
   ```
   nmap -sU <target>
   ```

8. Use uncommon TCP flag combinations:
   ```
   nmap -sN <target>  # NULL scan
   nmap -sF <target>  # FIN scan
   nmap -sX <target>  # Xmas scan
   ```

9. Idle scan (-sI):
   ```
   nmap -sI <zombie_host> <target>
   ```

10. Use scripts to manipulate packets:
    Various NSE scripts can be used for this purpose.

## 10. Example Advanced Scan
```bash
sudo nmap 10.129.128.167 -Pn --disable-arp-ping -n -D RND:10 -sV -p- --source-port 53 -vv --dns-servers 10.129.128.167fping
```


### Debugging:
|                  |                                                          |
| ---------------- | -------------------------------------------------------- |
| `--packet-trace` | Shows all packets sent and received                      |
| `--reason`       | shows reason for why a host/port is filtered/closed/open |
