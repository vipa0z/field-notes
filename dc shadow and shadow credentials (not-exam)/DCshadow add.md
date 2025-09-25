
# DCShadow Attacks

DCShadow attacks are difficult to prevent. Like [DCSync](https://blog.netwrix.com/2021/11/30/what-is-dcsync-an-introduction/), it does not abuse a vulnerability that could be patched; it exploits valid and necessary functions of the replication process, which cannot be turned off or disabled. DCShadow attacks are also difficult to detect, since the changes that the adversary requests are registered, processed and committed as legitimate domain replication.


Here is an overview of the steps in a DCShadow attack:

1. An attacker obtains Domain Admin or Enterprise Admin permissions, for example, by compromising a poorly secured group-managed service account.
2. The attacker uses DCShadow to register a computer object (such as a workstation) as a domain controller by making changes to the [AD](https://www.netwrix.com/what_is_active_directory_ebook.html "https://www.netwrix.com/what_is_active_directory_ebook.html") configuration schema and the workstation’s SPN. Now, AD thinks the workstation is a DC so it’s trusted to replicate changes.
3. The attacker submits changes for replication, such as changes to password data, account details or security group membership. Once replication is triggered, changes are published and committed by the other DCs.
4. The attacker uses DCShadow to remove the rogue domain controller from the [AD database](https://blog.netwrix.com/2017/02/17/active-directory-database/) to cover their activity.
