find all functionalities, inspect their parameters, 
test all and they all seam to have the same request

![](Pasted%20image%2020250601163039.png)

lets reverse how these parameters will be translated to bash inside the function
hmm, that my translate to something like
```
mv file1 file2
or
mv 23525424.txt 5525254.txt
```
so i think it  would be better to inject our characters at `file2`
or the request parameter called `to`,

if we try injecting different characters to find out  which are allowed,  `%26%26` seams to be our winner as it's not being blocked.
if we try to inject the raw command, its ofcourse blocked so lets try the bypass techniques taught in the module.
after trying different linux techniques the only one that worked was base64 encoding and decoding with bash  as follows:
```
└─$ echo -n 'cat /flag.txt' | base64
Y2F0IC9mbGFnLnR4dA==
```

```
bash<<<$(base64 -d<<<Y2F0IC9mbGFnLnR4dA==)
```
now if we combine the `%26%26` character with payload above, the request is going to look like this:

![](Pasted%20image%2020250601162954.png)
