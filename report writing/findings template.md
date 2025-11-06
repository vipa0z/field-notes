
Help me Draft a finding for that goes into my penetration test report, use professional but not overly formal tone,  i will provide you with the finding information, your task is to help me complete whichever is missing, avoid using astriks anywhere in the report, avoid using headers aside from the ones mentioned(only section titles like in the template)
## Finding title:
### <title>:
CVSS: 
CWE:

---

### Overview:


---
### description inc Root Cause
paragraph overview of the vulnerability type, relevance

---
### Impact
paragraph detailing impact if the vulnerability was to be discovered by attackers (dont be generic here, be specific, also be consice)

### Recommendation
paragraph with or without bullets for remediations, no more than 4 bullets, add a note if the configuration require testing first or can be damaging if directly applied

---
### Evidence
detailed reproduction steps

placeholder for  screenshot



---

### Attack chain summary 
(if it got me to internal network add it)



---
### Remediation Summary
asseses whether the remediations are long,medium, or short term, can be both or all and write them as a reference section
example: 
short term:
<finding name> - do xyz
medium term:
<finding name> - do xyz
long term:
<finding name> - do xyz

---

### Compromised hosts/vhosts
provide a single cell that can be appended to my compromised VHOSTs appendix.
ex: host/vhost| scope| method used| cleanup| md5|
where the scope is either external or internal, cleanup is notes about what files were uploaded to the host for defenders to remove later, md5 is the hash of the file.
---

### Compromised users


---
### Cleanup
webshell on target left at page X
