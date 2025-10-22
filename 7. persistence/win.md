
1. add user  
```
net user /add:vipa0z Password123!
```
2. make the user member of domain admins/ local 'Administrators'
```
net localgroup Administrators /add:vipa0z
```


3. scheduled tasks/startup scripts, batch files...etc
4. C2 BOFs (TBA)