i'm writing many findings that i have found in my pentesting assesment and im now in the phase of writing a report for the client, for each vulnerability i prompt you with,  i want you to create a finding entry for it. here is the description for what i may pass to you or may not, if i dont, you automatically create these entries, if i do tho, you should write them as is if no grammer/wording you could do is better. the final output for the finding should be as like this:
#### Title:
descriptive title
#### cvvs4.0 score:
you type this,  but i may give you a high,critical or low,info (you should adjust the rating so it aligns with me)
#### cwe:
the correct cwe for that type of vulnerability ( i may give you psuedo one but be sure it actually exists first by looking it up)
#### root cause:
short summary on intended functionality, and the vulnerability, and context on which part is vulnerable (what and why)
#### Impact:
what's the impact of this vulnerability? try to find the most common impact and try to avoid mixing up with other types ( do not cook)
#### Remediation:
Describe the remediation for this vulnerability 
#### Evidence:
i will provide you with a placeholder for screenshot or describe you what i need as steps and captions.
#### Remediation summary:
i will suggest either short,medium, or long term remediation for this finding or you decide which remediation to do for that finding and which category: short med or long term.

#### exploited host table row entry:
host|scope: external/internal| method| notes|
for example:
|vhost/ip/hostname|external/internal| file upload to RCE|any notes(exploit file upload to RCE)