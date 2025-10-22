https://0xffsec.com/handbook/shells/restricted-shells/
https://gtfobins.github.io/#+shell%20
## FIRST: Find all Installed Packages and Look for  GTFOBIN binaries.
` Match  package list with GFTOBINS`
```shell-session
$ for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done

Check GTFO for: ab                                         
Check GTFO for: apt                                        
Check GTFO for: ar                                         
Check GTFO for: as         
Check GTFO for: ash                                        
Check GTFO for: aspell                                     
Check GTFO for: at     
Check GTFO for: awk      
```

- If your user can run a binary with `sudo` (check with `sudo -l`), and it’s in GTFOBins with a `Sudo` section, then you can usually escalate to root.
- If a binary has the **SUID bit** set (`-rwsr-xr-x root root`), and it’s in GTFOBins under `SUID`, then you can often abuse it to escalate to root.
- **capabilities** if a binary is **capabilities-enabled** (`setcap`), or runs as a service with higher privileges.

#### Command injection
Imagine that we are in a restricted shell that allows us to execute commands by passing them as arguments to the `ls` command. Unfortunately, the shell only allows us to execute the `ls` command with a specific set of arguments, such as `ls -l` or `ls -a`, but it does not allow us to execute any other commands. In this situation, we can use command injection to escape from the shell by injecting additional commands into the argument of the `ls` command.

For example, we could use the following command to inject a `pwd` command into the argument of the `ls` command:


```shell-session
ls -l `pwd` 
```

This command would cause the `ls` command to be executed with the argument `-l`, followed by the output of the `pwd` command. Since the `pwd` command is not restricted by the shell, this would allow us to execute the `pwd` command and see the current working directory, even though the shell does not allow us to execute the `pwd` command directly.

#### Command Substitution

Another method for escaping from a restricted shell is to use command substitution. This involves using the shell's command substitution syntax to execute a command. For example, imagine the shell allows users to execute commands by enclosing them in backticks (`). In that case, it may be possible to escape from the shell by executing a command in a backtick substitution that is not restricted by the shell.

#### Command Chaining

In some cases, it may be possible to escape from a restricted shell by using command chaining. We would need to use multiple commands in a single command line, separated by a shell metacharacter, such as a semicolon (`;`) or a vertical bar (`|`), to execute a command. For example, if the shell allows users to execute commands separated by semicolons, it may be possible to escape from the shell by using a semicolon to separate two commands, one of which is not restricted by the shell.

#### Environment Variables

For escaping from a restricted shell to use environment variables involves modifying or creating environment variables that the shell uses to execute commands that are not restricted by the shell. For example, if the shell uses an environment variable to specify the directory in which commands are executed, it may be possible to escape from the shell by modifying the value of the environment variable to specify a different directory.

#### Shell Functions

In some cases, it may be possible to escape from a restricted shell by using shell functions. For this we can define and call shell functions that execute commands not restricted by the shell. Let us say, the shell allows users to define and call shell functions, it may be possible to escape from the shell by defining a shell function that executes a command.