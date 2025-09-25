### At the start of the exam
---------------------------x-------------------------------------------------------------------------------------

- at the start fill out email and contact info
- Meta
    - **Candidate name, title and email**
    - **Engagement Information**
- Document Control
    - **Customer Contacts**
- Network Penetration Testing Assessment Summary
    - Network Summary
- Executive Summary    - **Scope**
---
### -  discovered a host and scanned ports via nmap
**Host & Service Discovery**
------x-------------------------------------------------x-----------------------------------------------------------------x


---
### -  discovered subdomains and vhosts
------x-------------------------------------------------x-----------------------------------------------------------------x

---
### When A Security Finding Is Discovered 

For each security finding, you need to add a **Finding** to the report.
------x-------------------------------------------------x-----------------------------------------------------------------x
Fill out all the fields:

- Findings
    - **Title**
    - **CWE**
    - **CVSS**
    - reference
    - **Overview**
    - **Impact**
    - **Affected Components**
    - **Recommendations**
    - **Details**
- Remediation Summary
    - **Short Term**
    - **Medium Term**
    - **Long Term**
    in the **Details** field, provide a step-by-step procedure to reproduce the discovery and exploitation of the finding, including explanations for each step. Be sure to include:
- Plenty of images and code blocks to illustrate your points.
- Label all images and code blocks clearly.
- Highlight important lines in code blocks.
- Crop images to show only the relevant portions.
- Redact any sensitive information from both images and code blocks.
- Mention tools and scripts used.

---
### When You Get Foothold On A Host

You’ll also document the steps for this initial compromise in the compromise walkthrough.
------x-------------------------------------------------x-----------------------------------------------------------------x
- Internal Network Compromise Walkthrough
    - Walkthrough Summary
    - **Detailed Walkthrough**
- Appendix
    - **Exploited Hosts**
    - host:
For the **Exploited Hosts** appendix, fill in a table with the following columns for each host:

| Column | Explanation                                                                                                              |
| ------ | ------------------------------------------------------------------------------------------------------------------------ |
| Host   | The hostname and IP address of the host. E.g., “`HOST01` (`192.158.1.38`)”                                               |
| Scope  | Indicate either “External” or “Internal,” depending on whether the host is external-facing or internal (e.g., in a DMZ). |
| Method | Describe the exploitation method used to compromise the host. E.g., “Command injection on web application.”              |
| Notes  | Include any additional notes about the host. E.g., “Domain Controller for target.htb.”                                   |


For the compromise walkthrough, start by reviewing the **Walkthrough Summary** field. Make any necessary adjustments and see if it looks good to you. Feel free to rephrase it in your own words if needed, but otherwise you can leave it as is.

For the **Detailed Walkthrough**, you’ll document the steps related to the host you just exploited.

Use the following structure, which is divided into two parts:

```
{{ report.candidate.a_name }} ("the tester" herein) performed the following to fully compromise the <DOMAIN> domain.

<Here, include a numbered bullet point list where each point outlines a step in the attack chain.>

**Detailed reproduction steps for this attack chain are as follows:**

<Here, provide detailed steps in a writeup format.>
```

For both parts, exclude any findings unrelated to the shortest attack path to domain compromise. Write in the third person, referring to yourself as “the tester.”

- **Bullet Point List:** Keep it concise and focused. It’s fine to have multiple points for a single finding, but don’t overdo it. Don’t include images, code blocks, or any extraneous details.
    
- **Detailed Walkthrough:** Describe the steps thoroughly in a writeup format. Be highly descriptive and explain each technique or action in detail, at least once. Include plenty of images and code blocks—when in doubt, err on the side of including too many rather than too few. Add pertinent comments where appropriate, such as acknowledging effective security measures.
    

As you compromise more hosts during the engagement, this field will gradually fill up, resulting in a complete walkthrough of the attack chain by the end.

---
### When You Root A Host
Once you’ve gained the highest local privilege on a host (root or SYSTEM), 
------x-------------------------------------------------x-----------------------------------------------------------------x
fill out the following fields:

- Internal Network Compromise Walkthrough
    - **Detailed Walkthrough**
- Appendix
    - **Changes/Host Cleanup**
First, take a moment to recall all the changes you made to the host since gaining a foothold. This could include actions like adding a web shell, transferring an executable, creating a local user, etc.

Even if you reverted a change, such as deleting a PoC script after using it, it’s important to document it in the report. The transfer might have triggered an EDR alert, and the client will need to know whether it was part of the pentest or a potential breach.

After compiling your list, add a table to the **Changes/Host Cleanup** appendix with the following columns:

|Column|Explanation|
|---|---|
|Host|The hostname and IP address of the host. E.g., “`HOST01` (`192.158.1.38`)”|
|Scope|Indicate either “External” or “Internal,” depending on whether the host is external-facing or internal (e.g., in a DMZ).|
|Change/Cleanup Needed|A description of the change made. If it involves creating or transferring a file, include the MD5 sum. E.g., “Uploaded a web shell to `/var/www/html/shell.php`. MD5: `6ea3393fd7a4544d2fda0e42d7634013`”|

Make sure to add a heading before the table to identify which host the changes relate to.

---
### When You Compromise A User

Once you gain control of a user, whether by obtaining their passwords, NTLM hash or however else, add this information to the appropriate appendix.
------x-------------------------------------------------x-----------------------------------------------------------------x
- Appendix
    - **Compromised Users**

I recommend creating one table for local users and another for each Active Directory domain where you’ve compromised a user. Alternatively, you could create a separate table for local users on each individual host if that works better for you.

The tables should include the following columns:

|Column|Explanation|
|---|---|
|Username|The username for the user. E.g., “`www-data`”|
|Method|The method you used to gain control over this user. E.g., “Kerberoasted”|
|Notes|Notes about the user. For local users, specify the host they belong to. If the user has a specific purpose, mention that too. E.g., “`HOST01`’s local user for Tomcat service.”|

As always, ensure that tables are preceded by a heading identifying the host or domain they are associated with.

---
### When You Capture A Flag
You captured a flag—great job! Now, add it to the appropriate appendix:
------x-------------------------------------------------x-----------------------------------------------------------------x
- Appendix
    - **Flags Discovered**

I recommend using a single table with the following columns:

|Column|Explanation|
|---|---|
|Flag #|The flag number. Sort the rows in ascending order. E.g., “9.”|
|Host|The hostname of the host where the flag was found. E.g., “`HOST01`”|
|Flag Value|The exact value of the flag as found.|
|Flag Location|Where the flag was found. If it was in a file, provide the full path. E.g., “`/root/flag.txt`”|
|Method Used|The method used to exploit your way to the flag. E.g., “Privilege escalation via sudo rights over the `wget` binary.”|

---
### When The Engagement Is Done
------x-------------------------------------------------x-----------------------------------------------------------------x
- Executive Summary
    - **Assessment Overview and Recommendations**
- Remediation Summary
    - **Short Term**
    - **Medium Term**
    - **Long Term**
- Appendix
    - **Domain Password Review**