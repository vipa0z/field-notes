| id  | description                                            |
| --- | ------------------------------------------------------ |
| 1   | export certs and private keys with windows crypto APIs |
| 2   | export certificates and private keys using DPAPI       |
Here are the techniques organized into "Offensive Techniques" and "Defensive Techniques" tables:

| ID        | Description                                                                                                                                                     |
| :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| THEFT3    | Extracting machine certificates and private keys using DPAPI                                                                                                    |
| THEFT4    | Theft of existing certificates via file/directory triage                                                                                                        |
| THEFT5    | Using the Kerberos PKINIT protocol to retrieve an account’s NTLM hash                                                                                           |
| PERSIST1  | Account persistence via requests for new authentication certificates for a user                                                                                 |
| PERSIST2  | Account persistence via requests for new authentication certificates for a computer                                                                             |
| PERSIST3  | Account persistence via renewal of authentication certificates for a user/computer                                                                              |
| ESC1      | Domain escalation via No Issuance Requirements + Enrollable Client Authentication/Smart Card Logon OID templates + CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT            |
| ESC2      | Domain escalation via No Issuance Requirements + Enrollable Any Purpose EKU or no EKU                                                                           |
| ESC3      | Domain escalation via No Issuance Requirements + Certificate Request Agent EKU + no enrollment agent restrictions                                               |
| ESC4      | Domain escalation via misconfigured certificate template access control                                                                                         |
| ESC5      | Domain escalation via vulnerable PKI AD Object Access Control                                                                                                   |
| ESC6      | Domain escalation via the EDITF_ATTRIBUTESUBJECTALTNAME2 setting on CAs + No Manager Approval + Enrollable Client Authentication/Smart Card Logon OID templates |
| ESC7      | Vulnerable Certificate Authority Access Control                                                                                                                 |
| ESC8      | NTLM Relay to AD CS HTTP Endpoints                                                                                                                              |
| DPERSIST1 | Domain persistence via certificate forgery with stolen CA private keys                                                                                          |
| DPERSIST2 | Domain persistence via certificate forgery from maliciously added root/intermediate/NTAuth CA certificates                                                      |
| DPERSIST3 | Domain persistence via malicious misconfigurations that can later cause a domain escalation                                                                     |

**Defensive Techniques**

| ID       | Description                                      |
| :------- | :----------------------------------------------- |
| PREVENT1 | Treat CAs as Tier 0 Assets                       |
| PREVENT2 | Harden CA settings                               |
| PREVENT3 | Audit Published templates                        |
| PREVENT4 | Harden Certificate Template Settings             |
| PREVENT5 | Audit NtAuthCertificates                         |
| PREVENT6 | Secure Certificate Private Key Storage           |
| PREVENT7 | Enforce Strict User Mappings                     |
| PREVENT8 | Harden AD CS HTTP Enrollment Endpoints           |
| DETECT1  | Monitor User/Machine Certificate Enrollments     |
| DETECT2  | Monitor Certificate Authentication Events        |
| DETECT3  | Monitor Certificate Authority Backup Events      |
| DETECT4  | Monitor Certificate Template Modifications       |
| DETECT5  | Detecting Reading of DPAPI-Encrypted Keys        |
| DETECT6  | Use Honey Credentials                            |
| DETECT7  | Miscellane (Note: description cut off in source) |