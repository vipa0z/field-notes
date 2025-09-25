### identifying hash types

Alternatively, [hashID](https://github.com/psypanda/hashID) can be used to quickly identify the hashcat hash type by specifying the `-m` argument.

```shell-session
hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'

Analyzing '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
[+] MD5 Crypt [Hashcat Mode: 500]
[+] Cisco-IOS(MD5) [Hashcat Mode: 500]
```
