 NTLMV2: 5600 ->  challenge response
NT: 1000 -> nt hash stored in SAM/NTDS.DIT`
LM: 3000 -> LM hash stored in SAM'
NTDS.DIT stores in  'NTLM ' format
TGS: 13100
ASREP: TBA

---
example NTLMV2 challenge response returned from responder:
you pass the whole thing to hashcat like in this format:
`NTLMV2 RESPONSE -> AB920-user.ntlmv2`
```
AB920::INLANEFREIGHT:2982b690b4275f4e:665F0EA9F65C10D7B6AAB6B96D077236:010100000000000080B7847A5608DC01EC92DF6A95BE17C40000000002000800560037004C00570001001E00570049004E002D00320039003000510054004A005300540041004C00360004003400570049004  
<SNIP>
```
#### NTDS.dit
Impacket / `secretsdump` (pwdump style): best tool to dump nt and crack directly btw
```
domain\user:RID:LMHASH:NTHASH:::
```
you crack the NThash portion






