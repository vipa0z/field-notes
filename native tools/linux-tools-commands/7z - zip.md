```

7z a -tzip -mx=0 -mhe=on -p  -mhe=on archiave.zip *
```

    7z a → add files to an archive

    -tzip → output as a .zip file

    -mx=0 → no compression (store mode)
`-mhe=on` encrypt metadata
    -p → set a password (you’ll be prompted to enter it)

    archive.zip → name of the output file

### 7z
extract zip archive
```
7z x filename.zip
```