
# usage of compression in corporate environment
Let us assume the role of an employee at an administrative company and imagine that a client requests a summary of an analysis in various formats, such as Excel, PDF, Word, and a corresponding presentation. One approach would be to send these files individually. However, if we extend this scenario to a large organization managing multiple simultaneous projects, this method of file transfer can become cumbersome and may result in individual files being misplaced. In such cases, employees often rely on archive files, which allow them to organize necessary documents in a structured manner (typically using subfolders) before compressing them into a single, consolidated file.


Besides standalone files, we will often run across `archives` and `compressed files`—such as ZIP files—which are protected with a password.

There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.


Note that not all archive types support native password protection, and in such cases, additional tools are often used to encrypt the files. For example, TAR files are commonly encrypted using `openssl` or `gpg`.
A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed).
print all types to terminal:
```shell-session
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

# zip cracking
```shell-session
$ zip2john ZIP.zip > zip.hash
cat zip.hash 
```
# Cracking openssl encrypted file 

When cracking OpenSSL encrypted files, we may encounter various challenges, including numerous false positives or complete failure to identify the correct password. To mitigate this, a more reliable approach is to use the `openssl` tool within a `for` loop that attempts to extract the contents directly, succeeding only if the correct password is found.
attempt to crack pass with wordlist using openssl:
```shell-session
$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```
Once the `for` loop has finished, we can check the current directory for a newly extracted 
```shell-session
ls

customers.csv  GZIP.gzip  rockyou.txt
```


# cracking bitlocker encrypted drives
[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows operating system.
it uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths. If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

In enterprise environments, virtual drives are sometimes used to store personal information, documents, or notes on company-issued devices to prevent unauthorized access. 

To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john` to [four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker):
```shell-session
$ bitlocker2john -i Backup.vhd > backup.hashes
```
the first two correspond to the BitLocker password, while the latter two represent the recovery key.Because the recovery key is very long and randomly generated, it is generally not practical to guess—unless partial knowledge is available. Therefore you should only focus on cracking the first hash of the 4, ``$bitlocker$0$...``
```
 grep "bitlocker\$0" backup.hashes > backup.hash
```


Once a hash is generated, either `JtR` or `hashcat` can be used to crack it. For this example, we will look at the procedure with `hashcat`. The hashcat mode associated with the `$bitlocker$0$...` hash is `-m 22100`. 

We supply the hash, specify the wordlist, and define the hash mode. Since this encryption uses strong AES encryption, cracking may take considerable time depending on hardware performance.
`bitlocker2john` usually produces lines like:

```shell-session
$ hashcat -a 0 -m 22100 '$bitlocker$0$4f<snip>8576$12$00b' /usr/share/wordlists/rockyou.txt
```

#### Mounting BitLocker-encrypted drives in Windows
After successfully cracking the password, we can access the encrypted drive.
double click on the `.vhd` and enter pass.
![](bitlocker.webp)
#### Mounting BitLocker-encrypted drives in Linux (or macOS)

To do this, we can use a tool called [dislocker](https://github.com/Aorimn/dislocker). First, we need to install the package using `apt`:
```shell-session
$ sudo apt-get install dislocker
```
prepare directories that the .`vhd` file will be mounted to:
```shell-session
$ sudo mkdir -p /media/bitlocker
$ sudo mkdir -p /media/bitlockermount
```
We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:
```shell-session
$ sudo losetup -f -P Backup.vhd
$ sudo dislocker /dev/<MOUNT-POINTlO0P1> -u<PASS> -- /media/bitlocker
$ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```
If everything was done correctly, we can now browse the files:
```shell-session
$ cd /media/bitlockermount/
ls -la
```