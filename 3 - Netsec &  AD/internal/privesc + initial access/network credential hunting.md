childitem
snaffler/snafflepy
manhunt

With all of this in mind, you may want to begin with basic command-line searches (e.g., `Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...`) before scaling up to more advanced tools. Let's take a look at how we can use `MANSPIDER`, `Snaffler`, `SnafflePy`, and `NetExec` to automate and enhance this
```shell-session
 nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```

tcpdump -i