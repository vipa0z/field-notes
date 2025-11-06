
The module `spider_plus` will dig through each readable share on the host and list all readable files. Let's give it a try.

```shell
sudo nxc smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'

```
connect to shares with smbclient-ng
```
smbclientng -H DC01.ad.someorg.local -d ad.someorg.local -u 'hporter' -p 'Gr8hambino!'  --host 172.16.8.3
```

## smbmap

list share recursively 
```shell-session
$ smbmap -u forend -p Klmcargo2 -d ad.someorg.local -H 172.16.5.5  -r <share>
```
### Recursive list of all directories (no files)
```shell-session
$ smbmap -u forend -p Klmcargo2 -d ad.someorg.local -H 172.16.5.5 -r 'Department Shares' --dir-only
```
## lists dir
```
smbmap -r
```
also at [pillaging](../../p)

## group policy files in SYSVOL
#group-policy #gpp-decrypt
look for groups,xml in sysvol
get cpassword
```
gpp-decrypt <encrypted>
```

## Snaffler

[Snaffler](https://github.com/SnaffCon/Snaffler) is a tool that can help us acquire credentials or other sensitive data in an Active Directory environment. Snaffler works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories. Once that is done, it iterates through any directories readable by our user and hunts for files that could serve to better our position within the assessment. Snaffler requires that it be run from a domain-joined host or in a domain-user context.

```bash
Snaffler.exe -s -d domain.local -o snaffler.log -v data
```

```

## sysvol

- [ ] looked for `groups.xml` file

## file search