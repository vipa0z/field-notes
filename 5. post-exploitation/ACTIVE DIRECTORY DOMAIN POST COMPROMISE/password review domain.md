1. get ntds dump from secrets dump
2. use hashcat -m 1000 on that file 
3. type --show >> hashes.cpot
4. generate report with dpat
```
dpat.py -n ntds.dit -c hashcat.cpot
```