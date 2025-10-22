
## vhosts
```shell-session
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

filter response by size with `-fs`
filter status codes with `-fc`

once you discover subdomains, add them to host file

```

remove some some fluff and get full url
```
ffuf -u http://inlanefreight.htb:35915/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ  -recursion -recursion-depth 1 -e .php -v
```

## get parameter fuzzing
**Tip:** Fuzzing parameters may expose unpublished parameters that are publicly accessible. Such parameters tend to be less tested and less secured, so it is important to test such parameters for the web vulnerabilities we discuss in other modules.
wordlist: `/seclists/Discovery/Web-Content/burp-parameter-names.txt`. With that, we can run our scan.
```shell-session
 ffuf -w /burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

### fuzzing post body
```shell-session
ffuf -w /opt/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```


## fuzzing parameter values
When it comes to fuzzing parameter values, we may not always find a pre-made wordlist that would work for us, as each parameter would expect a certain type of value.

-------------------
To add a slash (/) after every request, we can use the **-f** or **–add-slash** flag.


## Parameter Value Fuzzing
###  picking Wordlists for specific parameters
When it comes to fuzzing parameter values, we may not always find a pre-made wordlist that would work for us, as each parameter would expect a certain type of value.
## paramter  Key fuzzing
- try parameter fuzzing with GET and POST
- for post u need to set the content type header  and provide -d flag 
```shell-session
 -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```


For some parameters, like usernames, we can find a pre-made wordlist for potential usernames, or we may create our own based on users that may potentially be using the website. For such cases, we can look for various wordlists under the `seclists` directory and try to find one that may contain values matching the parameter we are targeting. In other cases, like custom parameters, we may have to develop our own wordlist. In this case, we can guess that the `id` parameter can accept a number input of some sort. These ids can be in a custom format, or can be sequential, like from 1-1000 or 1-1000000, and so on. We'll start with a wordlist containing all numbers from 1-1000.



```shell-session
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```
