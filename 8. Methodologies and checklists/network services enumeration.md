- [ ] attempt anonymous login
- [ ]  attempt a banner grab on  each service  nc -nv 10.129.203.101 22
- [ ] search exploits for that specific version
- [ ] i
### FTP
- banner grab and look for CVEs that's not a DOS
- check to see if you can perform anonymous access auth
- check read acess, what files can you access
- check if you can write files, often you can write webshells 
```
passive off
put hi.txt
```
### SMTP
- check if vrfy expn,.etc enabled
- use smtp-user-enum tool to enumerate users
### imap/pop3/
- [ ] attempt anonymous login
- [ ] look for emails/messagse
- [ ] check if vrfy and alike are enabled, if enabled do user  enumeration
- [ ] user enum 
### rpcbind
- [ ] allows enumerating OS
- [ ]  `rpcbind` service which should not be exposed externally, so we could write up a `Low` finding for `Unnecessary Exposed` 