## Group Policy Preferences (GPP) Passwords

When a new GPP is created, an .xml file is created in the SYSVOL share, which is also cached locally on endpoints that the Group Policy applies to. These files can include those used to:

- Map drives (drives.xml)
- Create local users
- Create printer config files (printers.xml)
- Creating and updating services (services.xml)
- Creating scheduled tasks (scheduledtasks.xml)
- Changing local admin passwords.

modules we can use for finding gpp files
```shell-session
 nxc smb -L | grep gpp

[*] gpp_autologin             Searches the domain controller for registry.xml to find autologon information and returns the username and password.
[*] gpp_password              Retrieves the plaintext password and other information for accounts pushed through Group Policy Preferences.
```

example
```shell-session
$ crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin
```

 The `cpassword` attribute value is AES-256 bit encrypted, but Microsoft [published the AES private key on MSDN](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN), which can be used to decrypt the password. 
 
 Any domain user can read these files as they are stored on the SYSVOL share, and all authenticated users in a domain, by default, have read access to this domain controller share.

This was patched in 2014 [MS14-025 Vulnerability in GPP could allow elevation of privilege](https://support.microsoft.com/en-us/topic/ms14-025-vulnerability-in-group-policy-preferences-could-allow-elevation-of-privilege-may-13-2014-60734e15-af79-26ca-ea53-8cd617073c30), to prevent administrators from setting passwords using GPP. The patch does not remove existing Groups.xml files with passwords from SYSVOL. If you delete the GPP policy instead of unlinking it from the OU, the cached copy on the local computer remains.

f you retrieve the cpassword value more manually, the `gpp-decrypt` utility can be used to decrypt the password as follows:

#### Decrypting the Password with gpp-decrypt

Miscellaneous Misconfigurations

```shell-session
magdy3660@htb[/htb]$ gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE

Password1
```