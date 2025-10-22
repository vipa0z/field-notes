
wordlist: `/seclists/Discovery/Web-Content/burp-parameter-names.txt`.
With that, we can run our scan.

### Get Parameter Fuzzing

- Fuzz parameter names (keys):
```
ffuf -u "http://target.com/page.php?FUZZ=1" -w /path/to/param_names.txt
```
- Fuzz parameter values for a known parameter (e.g., `id`):

```
ffuf -u "http://target.com/page.php?id=FUZZ" -w /path/to/param_values.txt
```

- Fuzz multiple parameters by placing `FUZZ` in each parameter:
```
ffuf -u "http://target.com/page.php?user=FUZZ&pass=FUZZ" -w /path/to/wordlist.txt -t 50
```

### 2. **Fuzzing POST parameters and their values**

Using **ffuf**:

- Fuzz parameter names in POST data (if parameter name is unknown):

```shell-session
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
``````

- Fuzz parameter values for a known parameter (e.g., `KEY`):

```
ffuf -u http://target.com/login.php -X POST -d "<KEY>=FUZZ&password=pass123" -w /path/to/param_values.txt
```

- Fuzz multiple POST parameters (only one `FUZZ` per request):

```
ffuf -u http://target.com/login.php -X POST -d "user=FUZZ&pass=FUZZ" -w /path/to/wordlist.txt
```

But to fuzz multiple parameters independently, run multiple ffuf instances or use complex scripting.

---







-------------- DEPRECATED NOTES-----------------------------------
```shell-session
 ffuf -w /burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

### fuzzing post body
```shell-session
ffuf -w /opt/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

