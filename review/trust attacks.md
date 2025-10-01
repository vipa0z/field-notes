
trust relationships
bidirectional <->
transitive A->B->C
	A -> C ( A trusts C now, transfered from B)

One-way?

Cross-Forest

-----
### transiitve trust:
means the trust is extended to objects that the child domain trusts. 

### parent child
parent-child  inlanefrieght (parent) trusts <->corp.inlanefreight.local (child)
![[Pasted image 20250930200330.png]]
#### Trust Table Side By Side

|Transitive|Non-Transitive|
|---|---|
|Shared, 1 to many|Direct trust|
|The trust is shared with anyone in the forest|Not extended to next level child domains|
|Forest, tree-root, parent-child, and cross-link trusts are transitive|Typical for external or custom trust setups|

---
### Cross-Link
trust between child domains to speed authentication process

---
### tree-root
two way between forest root domain and tree root domain

---
### Forest Trust
a transitive trust between two forest root domains

---
### ESAE 
bastion forest for managing active directory

---
### External trust
Non transitive one way trust between two seperate domains in seperate forests
(forest-trust), utilizes sidFIltering for requests not from domain


mental notes:
1. can i identify the trust relationship? ill use bloodhound and powerview

2. are the domains in the trust mapping graph in the same forest? or not? if not that could  mean the trust relationship is external one way

---
### one way and two way trusts ( one-way | bidirectional)
- `One-way trust`: Users in a `trusted` domain can access resources in a trusting domain, not vice-versa.
- `Bidirectional trust`: Users from both trusting domains can access resources in the other domain. For example, in a bidirectional trust between `INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL`, users in `INLANEFREIGHT.LOCAL` would be able to access resources in `FREIGHTLOGISTICS.LOCAL`, and vice-versa.

## attacking domain trusts
Domain trusts are often set up incorrectly and can provide us with critical unintended attack paths. Also, trusts set up for ease of use may not be reviewed later for potential security implications if security is not considered before establishing the trust relationship. A Merger & Acquisition (M&A) between two companies can result in bidirectional trusts with acquired companies, which can unknowingly introduce risk into the acquiring company’s environment if the security posture of the acquired company is unknown and untested. 

imagine you can kerberoast the second domain to obtain admin credentials for second domain, if trust is enabled and misconfigured you're literally domain admin in the principal domain/parent domain.