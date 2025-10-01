## Exchange Related Group Membership

A default installation of Microsoft Exchange within an AD environment (with no split-administration model) opens up many attack vectors, as Exchange is often granted considerable privileges within the domain (via users, groups, and ACLs).

### Exchange Windows Permissions Group

The group `Exchange Windows Permissions` is not listed as a protected group, but members are granted the ability to write a DACL to the domain object. This can be leveraged to give a user DCSync privileges.

**Attack Vectors:**

- An attacker can add accounts to this group by leveraging a DACL misconfiguration (possible)
- By leveraging a compromised account that is a member of the Account Operators group
- It is common to find user accounts and even computers as members of this group
- Power users and support staff in remote offices are often added to this group, allowing them to reset passwords

**Reference:** [Exchange-AD-Privesc GitHub Repository](https://github.com/gdedrouas/Exchange-AD-Privesc) - Details techniques for leveraging Exchange for escalating privileges in an AD environment.

### Organization Management Group

The Exchange group `Organization Management` is another extremely powerful group (effectively the "Domain Admins" of Exchange) and can access the mailboxes of all domain users.

**Key Points:**

- It is not uncommon for sysadmins to be members of this group
- This group also has full control of the OU called `Microsoft Exchange Security Groups`
- Contains the group `Exchange Windows Permissions`

## PrivExchange Attack

The `PrivExchange` attack results from a flaw in the Exchange Server `PushSubscription` feature, which allows any domain user with a mailbox to force the Exchange server to authenticate to any host provided by the client over HTTP.

### Technical Details

- The Exchange service runs as SYSTEM and is over-privileged by default
- Has WriteDacl privileges on the domain (pre-2019 Cumulative Update)
- This flaw can be leveraged to relay to LDAP and dump the domain NTDS database
- If we cannot relay to LDAP, this can be leveraged to relay and authenticate to other hosts within the domain
- **This attack will take you directly to Domain Admin with any authenticated domain user account**

## Exchange Servers as High-Value Targets

### Compromise Impact

If we can compromise an Exchange server, this will often lead to Domain Admin privileges. Additionally, dumping credentials in memory from an Exchange server will produce 10s if not 100s of cleartext credentials or NTLM hashes.

### Why This Happens

This is often due to users logging in to Outlook Web Access (OWA) and Exchange caching their credentials in memory after a successful login.

---

## Assessment Checklist

- [ ]  Enumerate Exchange Windows Permissions group membership
- [ ]  Check Organization Management group members
- [ ]  Assess Exchange server security posture
- [ ]  Test for PrivExchange vulnerability
- [ ]  Check Exchange version and patch level
- [ ]  Review Exchange-related DACL permissions
- [ ]  Test credential dumping from Exchange servers