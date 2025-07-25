---
layout: default
parent: Kerberos Delegation
title: Unconstrained Delegation
weight: 1
type: docs
---


### **Introduction**

In this section we will learn what's unconstrained delegation, the requirements for this attack is simple must know basic widows attack vector's and Kerberos communication for unconstrained delegation.
<br><br>

---

### **What's Unconstrained Delegation !!?**

Unconstrained delegation is a configuration in Microsoft Active Directory and Kerberos environments that allows a specifically designated service or computer to impersonate any authenticated user and access any resource in the domain on their behalf, by leveraging the user's Kerberos TGT.

Unconstrained delegation in Microsoft Active Directory can be assigned to the following object types:

- **Computer accounts**: Any domain-joined computer (servers or workstations) can be configured for unconstrained delegation. This is most common for application servers where certain legacy multi-tier applications require it.

- **User accounts**: Specifically, service accounts running under a user's security context can be delegated this setting. This is less common but is technically supported if the service requires impersonation capability

**Key points:**

- Only objects (computer or user accounts) with the **"Trusted for Delegation"** flag enabled can be used for unconstrained delegation.

- It is not assignable to groups, organizational units, or arbitrary AD objects—only to individual user and computer accounts.
<br><br>
---
### **Attack Vector**

- **Initial Access and Identification**:  
    Red teams first scout the environment via LDAP queries or enumeration tools to identify which computer or service accounts have unconstrained delegation enabled, excluding domain controllers since unconstrained delegation is enabled by default there.

- **Ticket Theft and Impersonation**:  
    Once a system with unconstrained delegation is compromised, attackers can monitor for incoming Kerberos Ticket Granting Tickets (TGTs) that users present when accessing that service. They steal or extract these TGTs from memory using commonly used tools like Rubeus or GhostPack. With these TGTs, red teamers can impersonate users fully—requesting access to any other resource in the domain without the user’s credentials.

- **Lateral Movement and Privilege Escalation**:  
    By impersonating privileged users such as Domain Admins, attackers can move laterally across the network, accessing sensitive systems and services to escalate privileges further and gain domain-wide control. This can include accessing critical systems like SCCM servers, which facilitate broader network compromise.

- **Persistence and Evasion**:  
    Red teams employ various persistence techniques post-compromise, such as backdooring scripts on compromised hosts, specifically targeting hosts with unconstrained delegation enabled as they provide high-value access

**Attack Workflow Summary**:

1. Identify hosts with unconstrained delegation enabled. 
2. Gain initial access (often via exploits or credential dumping).
3. Use tools to monitor and extract TGTs in memory.
4. Impersonate targeted users to request service tickets for additional resources.
5. Move laterally and escalate privileges, maintaining stealth and persistence.
<br><br>
---

### **Lab Setup !!!**

Comming Soon
