Enumeration is the key to privilege escalation. Several helper scripts (such as [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) and [LinEnum](https://github.com/rebootuser/LinEnum)) exist to assist with enumeration.

## LinEnum
https://github.com/rebootuser/LinEnum
## linpeas
## foothold commands
- `whoami` - what user are we running as
- `id` - what groups does our user belong to?
- `hostname` - what is the server named, can we gather anything from the naming convention?
- `ifconfig` or `ip a` - what subnet did we land in, does the host have additional NICs in other subnets?
- `sudo -l` - can our user run anything with sudo (as another user as root) without needing a password? This can sometimes be the easiest win and we can do something like `sudo su` and drop right into a root shell.
# os info
We'll start out by checking out what operating system and version we are dealing with.


## path
if the PATH variable for a target user is misconfigured we may be able to leverage it to escalate privileges. For now we'll note it down and add it to our notetaking tool of choice.
```shell-session
echo $PATH
```

## ENV
We can also check out all environment variables that are set for our current user, we may get lucky and find something sensitive in there such as a password.
```shell-session
$ env
```


## KERNEL Version
```shell-session
$ uname -a
```
## Users
All users on the system are stored in the `/etc/passwd` file. The format gives us some information, such as:
```shell-session
$ cat /etc/passwd | cut -f1 -d:
```
1. Username
2. Password
3. User ID (UID)
4. Group ID (GID)
5. User ID info
6. Home directory
7. - Shell
## /etc/passwd
Occasionally, we will see password hashes directly in the `/etc/passwd` file. This file is readable by all users, and as with hashes in the `/etc/shadow` file, these can be subjected to an offline password cracking attack.

list of hash algorithms on the password


|**Algorithm**|**Hash**|
|---|---|
|Salted MD5|`$1$`...|
|SHA-256|`$5$`...|
|SHA-512|`$6$`...|
|BCrypt|`$2a$`...|
|Scrypt|`$7$`...|
|Argon2|`$argon2i$`...|

We'll also want to check which users have login shells. Once we see what shells are on the system, we can check each version for vulnerabilities. Because outdated versions, such as Bash version 4.1, are vulnerable to a `shellshock` exploit.
```shell-session
grep "sh$" /etc/passwd
```
## groups
Each user in Linux systems is assigned to a specific group or groups and thus receives special privileges. For example, if we have a folder named `dev` only for developers, a user must be assigned to the appropriate group to access that folder.
```shell-session
$ cat /etc/group
```
list group members
```shell-session
$ getent group sudo
```

## CPU
information about the host itself such as the CPU type/version:

```shell-session
$ lscpu 
```

## SHELL TYPES
What login shells exist on the server? Note these down and highlight that both `Tmux` and `Screen `are available to us.

```shell-session
cat /etc/shells
```

## Protections
Often we will not have the privileges to enumerate the configurations of these protections but knowing what, if any, are in place, can help us not to waste time on certain tasks.
Some things to look for include:

- [Exec Shield](https://en.wikipedia.org/wiki/Exec_Shield)
- [iptables](https://linux.die.net/man/8/iptables)
- [AppArmor](https://apparmor.net/)
- [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
- [Fail2ban](https://github.com/fail2ban/fail2ban)
- [Snort](https://www.snort.org/faq/what-is-snort)
- [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)
## Drives
Next we can take a look at the drives and any shares on the system. If we discover and can mount an additional drive or unmounted file system, we may find sensitive files, passwords, or backups that can be leveraged to escalate privileges.
```shell-session
$ lsblk
```
#### Mounted File Systems


```shell-session
df -h
```
if we can extend our privileges to the `root` user, we could mount and read these file systems ourselves. Unmounted file systems can be viewed as follows:

```shell-session
cat /etc/fstab | grep -v "#" | column -t
```

# fstabs

 Can we mount an umounted drive and gain access to sensitive data? 
 Can we find any types of credentials in `fstab` for mounted drives by grepping for common words such as password, username, credential, etc in `/etc/fstab`?
 ```shell-session
$ cat /etc/fstab
```
# routing table
Check out the routing table by typing `route` or `netstat -rn`. Here we can see what other networks are available via which interface.
```shell-session
route
```

##  Recent communications with other hosts
We'll also want to check the arp table to see what other hosts the target has been communicating with.
```shell-session
arp -a
```

## Domain Environment
In a domain environment we'll definitely want to check `/etc/resolv.conf` if the host is configured to use internal DNS we may be able to use this as a starting point to query the Active Directory environment.
## printers
The command `lpstat` can be used to find information about any printers attached to the system. If there are active or queued print jobs can we gain access to some sort of sensitive information?


## Hidden files
`with regex` USE THIS ONE:
```
sudo find / -type f -name "*.sh" -exec grep -H "pass" {} + 2>/dev/null
```

`this one was provided in the module section but it didnt actually find anything useful...`
```shell-session
 find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep pass
```
## hidden diretories

```shell-session
find / -type d -name ".*" -ls 2>/dev/null
```
