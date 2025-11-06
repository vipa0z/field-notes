The following examples automate the synchronization of a remote domain controller's clock to initiate a corresponding zsh session.

```
faketime "$(rdate -n $DC_IP -p | awk '{print $2, $3, $4}' | date -f - "+%Y-%m-%d %H:%M:%S")" zsh
```

```
faketime "$(date +'%Y-%m-%d') $(net time -S $DC_IP | awk '{print $4}')"
```
