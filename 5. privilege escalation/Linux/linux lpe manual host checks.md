first, we'll answer the fundamental question: What operating system are we dealing with?


### Linpeas
- adm group (can read /var/logs)
- gtfo with file write/read, sudo
GTFO bins can be used to edit /etc/shadow 
read possible? read .ssh keys
- checlists online
- sudo
- kernel
- cronjobs
- processes
- local ports

#### Initial access commands:
- `whoami` - what user are we running as
- `id` - what groups does our user belong to?
- `hostname` - what is the server named, can we gather anything from the naming convention?
- `ifconfig` or `ip a` - what subnet did we land in, does the host have additional NICs in other subnets?
- `sudo -l` - can our user run anything with sudo (as another user as root) without needing a password? This can sometimes be the easiest win and we can do something like `sudo su` and drop right into a root shell.

## os
```shell-session
cat /etc/os-release
```

### path
if the PATH variable for a target user is misconfigured we may be able to leverage it to escalate privileges. For now we'll note it down and add it to our notetaking tool of choice.
```shell-session
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

```

#### check ENV
we may get lucky and find something sensitive in there such as a password. We'll note this down and move on.

```shell-session
env
```

### kernel version
We can do some searches to see if the target is running a vulnerable Kernel (which we'll get to take advantage of later on in the module) which has some known public exploit PoC
```shell-session
uname -a
```

### login shells, multiplexers
What login shells exist on the server? Note these down and highlight that both Tmux and Screen are available to us.
```shell-session
cat /etc/shells

/bin/sh
/bin/bash

/usr/bin/tmux
/usr/bin/screen
```

### defenses
We should also check to see if any defenses are in place and we can enumerate any information about them. Some things to look for include:

- [Exec Shield](https://en.wikipedia.org/wiki/Exec_Shield)
- [iptables](https://linux.die.net/man/8/iptables)
- [AppArmor](https://apparmor.net/)
- [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
- [Fail2ban](https://github.com/fail2ban/fail2ban)
- [Snort](https://www.snort.org/faq/what-is-snort)
- [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)
### DRIVES, mounted/shared file systems
we may find sensitive files, passwords, or backups that can be leveraged to escalate privileges.
```
lsblk
```

### printers
`lpstat` can be used to find information about any printers attached to the system.

#### fstab
check for mounted drives and unmounted drives. Can we mount an umounted drive and gain 

access to sensitive data? Can we find any types of credentials in `fstab` for mounted drives by grepping for common words such as password, username, credential, etc in `/etc/fstab`?

```shell-session
cat /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-BdLsBLE4CvzJUgtkugkof4S0dZG7gWR8HCNOlRdLWoXVOba2tYUMzHfFQAP9ajul / ext4 defaults 0 0
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/20b1770d-a233-4780-900e-7c99bc974346 /boot ext4 defaults 0 0
```
### routing table
what other networks are available?
```shell-session
route
```


## Internal DNS
In a domain environment we'll definitely want to check `/etc/resolv.conf` if the host is configured to use internal DNS we may be able to use this as a starting point to query the Active Directory environment.

We'll also want to check the arp table to see what other hosts the target has been communicating with.
```shell-session
arp -a

_gateway (10.129.0.1) at 00:50:56:b9:b9:fc [ether] on ens192
```

```

### shell vulnerabilities
We'll also want to check which users have login shells. Once we see what shells are on the system, we can check each version for vulnerabilities. Because outdated versions, such as Bash version 4.1, are vulnerable to a `shellshock` exploit.

```shell-session
 grep "sh$" /etc/passwd

root:x:0:0:root:/root:/bin/bash
```
## users
`check if hashes exist`
```shell-session
cat /etc/passwd
```
## hash blocks
hash blocks can help us to use and work with them later if needed. Here is a list of the most used ones:

| **Algorithm** | **Hash**       |
| ------------- | -------------- |
| Salted MD5    | `$1$`...       |
| SHA-256       | `$5$`...       |
| SHA-512       | `$6$`...       |
| BCrypt        | `$2a$`...      |
| Scrypt        | `$7$`...       |
| Argon2        | `$argon2i$`... |
`list of all users`
```shell-session
cat /etc/passwd | cut -f1 -d:

### groups
Each user in Linux systems is assigned to a specific group or groups and thus receives special privileges
```shell-session
 cat /etc/group
```

list members of a group
```shell-session
getent group sudo
```

### users home
check for history, ssh keys, creds, AD related creds..etc
```shell-session
ls /home
```


### config files

## mounted filesystems
`df -h`
#### Unmounted File Systems
```shell-session
cat /etc/fstab | grep -v "#" | column -t
```


### find all hidden files

```shell-session
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student
```
### hidden directories
```shell-session
find / -type d -name ".*" -ls 2>/dev/null
```

### Temporary logs, scripts
In addition, three default folders are intended for temporary files. These folders are visible to all users and can be read. In addition, temporary logs or script output can be found there. Both `/tmp` and `/var/tmp` are used to store data temporarily. However, the key difference is how long the data is stored in these file systems. The data retention time for `/var/tmp` is much longer than that of the `/tmp` directory. By default, all files and data stored in /var/tmp are retained for up to 30 days. In /tmp, on the other hand, the data is automatically deleted after ten days.

In addition, all temporary files stored in the `/tmp` directory are deleted immediately when the system is restarted. Therefore, the `/var/tmp` directory is used by programs to store data that must be kept between reboots temporarily.
```shell-session
ls -l /tmp /var/tmp /dev/shm
```
## checklists online
find suid binaries: read this blog
https://johnermac.github.io/notes/pnpt/linuxprivesc/#search-for-suid-files
```
find / -perm -4000 -type f  -ls 2>/dev/null

strace <file> 2>&1 | grep -i -E "open|access|somethin else" = trace executable file / the grep just filters
```
### Internal Enumeration
When we talk about the `internals`, we mean the internal configuration and way of working, including integrated processes designed to accomplish specific tasks.
### other checks
```
- What services and applications are installed?
    
- What services are running?
    
- What sockets are in use?
    
- What users, admins, and groups exist on the system?
    
- Who is current logged in? What users recently logged in?
    
- What password policies, if any, are enforced on the host?
    
- Is the host joined to an Active Directory domain?
    
- What types of interesting information can we find in history, log, and backup files
    
- Which files have been modified recently and how often? Are there any interesting patterns in file modification that could indicate a cron job in use that we may be able to hijack?
  
  

Current IP addressing information

Anything interesting in the /etc/hosts file?

Are there any interesting network connections to other systems in the internal network or even outside the network?

What tools are installed on the system that we may be able to take advantage of? (Netcat, Perl, Python, Ruby, Nmap, tcpdump, gcc, etc.)

Can we access the bash_history file for any users and can we uncover any thing interesting from their recorded command line history such as passwords?

```

### Services

## login time for users
It can also be helpful to check out each user's last login time to try to see when users typically log in to the system and how frequently. This can give us an idea of how widely used this system is which can open up the potential for more misconfigurations or "messy" directories or command histories.
```shell-session
 lastlog
```
### logged in users
```shell-session
w
```

### find history files
```shell-session
 find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```
### cronjobs
```shell-session
ls -la /etc/cron.daily/
```
packages
```shell-session
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```
binaries
```shell-session
ls -l /bin /usr/bin/ /usr/sbin/
```

gtfobins: for privesc
```shell-session
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```

strace
We can use the diagnostic tool `strace` on Linux-based operating systems to track and analyze system calls and signal processing. It allows us to follow the flow of a program and understand how it accesses system resources, processes signals, and receives and sends data from the operating system. In addition, we can also use the tool to monitor security-related activities and identify potential attack vectors, such as specific requests to remote hosts using passwords or tokens.

The output of `strace` can be written to a file for later analysis, and it provides a wealth of options that allow detailed monitoring of the program's behavior.
#### Trace System Calls
```shell-session
 strace ping -c1 10.129.112.20
```
#### Configuration Files

Linux Services & Internals Enumeration

```shell-session
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```
scripts
The scripts are similar to the configuration files. can be a treasure trove
#### Scripts

```shell-session
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```
#### Running Services by User
if it is a script created by the administrator in his path and whose rights have not been restricted, we can run it without going into the `root` directory.
```shell-session
ps aux | grep root
```