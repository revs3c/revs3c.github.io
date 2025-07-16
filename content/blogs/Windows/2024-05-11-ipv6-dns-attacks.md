---
layout: default
title:  "IPV6 DNS ATTACK"
weight: 20
type: docs
---

## #🤔What?   
   
The attack performed, that is ipv6 is a different than the LLMNR and SMB relay, likely same but different mechanisms before diving in let's consider knowing some technical term's.   
   
**#**🌐🆔6️⃣Wh**at is IPv6 ?   
> IPv6 stands for Internet Protocol version 6. It’s the most recent version of the Internet Protocol (IP), which is the system that identifies devices on a network and allows them to communicate with each other.   

   
### #🧭 What is WPAD?   
> WPAD stands for Web Proxy Auto-Discovery Protocol. It’s a method used by computers to automatically find out if they should use a proxy server for web access—and if so, which one.   

   
### #🏢 How does WPAD work in Windows Active Directory environments?   
> In a Windows Active Directory (AD) setup, WPAD is often used to simplify the configuration of proxy settings across a large number of computers.   

   
**Here’s how it typically works:**   
  - **A client device boots up and looks for proxy settings.**   
  - It tries to find a file called `wpad.dat`, which contains the proxy info.   
  - This file can be discovered via:   
      - **DNS** (e.g., `http://wpad.company.local/wpad.dat`)   
      - **DHCP** (if configured that way)   
  - If found, the client uses the rules in `wpad.dat` to decide when and how to use a proxy.   
   
     
### #🧾 What is LDAP?   
   
- **LDAP** stands for **Lightweight Directory Access Protocol**.It’s a protocol (a set of rules) used to **access and manage directory information**—like user accounts, groups, email addresses, devices, and more—usually in a **centralized directory service** like **Active Directory**.   
   
   
## #🛠️Working of IPv6 Attack.   
   
### 🔹 1. Environment Setup   
   
  - **Goal:** Exploit IPv6 preference + NTLM authentication to gain unauthorized access.   
  - Tools:   
      - `mitm6` — Spoofs IPv6 DNS/DHCP   
      - `ntlmrelayx` — Captures and relays NTLM hashes   
   
         
### 🔹 2. IPv6 DNS/DHCP Spoofing (mitm6)   
   
  - Windows prefers IPv6 over IPv4 → listens to attacker’s fake DNS server.   
  - You spoof DNS and become the default DNS for the victim.   
   
     
### ✅ Command:   
   
  ```
bash

CopyEdit
sudo mitm6 -i vboxnet0 -d tonystark.local
```
   
  > Replace vboxnet0 with your interface and tonystark.local with your domain.   

### 🔹 3. WPAD Hijacking via DNS Spoofing   
   
  - Victim sends a DNS request for `wpad.tonystark.local`.   
  - Attacker’s fake DNS responds with the attacker’s IP.   
   
  > 📌 This exploits the WPAD (Web Proxy Auto-Discovery Protocol) mechanism.   

###    
### 🔹 4. Serving Malicious Proxy Script (wpad.dat)   
   
  - Victim requests:   
      ```
http://wpad.tonystark.local/wpad.dat

```
  - Attacker hosts a fake `wpad.dat`, which sets them as an HTTP proxy.   
   
  > 📌 This configures the victim to send HTTP traffic (and NTLM auth) through the attacker.   

###    
### 🔹 5. NTLM Authentication Triggered   
   
  - Victim authenticates to the rogue proxy using NTLMv2.   
  - Attacker captures the NTLM challenge-response pair (without password).   
   
### 🔹 6. NTLM Relay to Target Service (ntlmrelayx)   
   
  - Captured NTLM is relayed to a real service, like LDAPS.   
  - If signing isn’t enforced → successful authentication.   
   
### ✅ Command:   
   
  ```
sudo ntlmrelayx -6 -t ldaps://192.168.56.109 -wh fakewpad.tonystark.local -l lootme

```
   
  - `-6`: Enable IPv6 listener   
  - `-t`: Target to relay to (LDAPS here)   
  - `-wh`: Hostname for WPAD server   
  - `-l`: Loot directory for dumping results   
   
## #🧐Why this attack works   
   
### 🔻 1. IPv6 is Enabled by Default   
   
  - Even if the network doesn’t actively use IPv6, it’s **still enabled** on Windows systems.   
  - This allows `mitm6` to **inject rogue DHCPv6 and DNS information**, hijacking the victim’s DNS config.   
   
  > 📌 No IPv6 usage ≠ no IPv6 traffic. That’s the trap.   

   
### 🔻 2. No DNSSEC or DHCPv6 Authentication   
   
  - Windows trusts DNS/DHCPv6 responses **without verifying authenticity**.   
  - There’s **no DNS server validation** in most internal networks.   
   
  > 📌 This makes spoofing as simple as replying faster than the real server.   

   
### 🔻 3. WPAD Protocol is Enabled   
   
  - WPAD (Web Proxy Auto-Discovery) is **on by default** in many setups.   
  - Systems automatically try to find proxy settings via `http://wpad.domain.local`.   
   
  > 📌 Legacy feature designed for convenience, now abused for credential capture.   

   
### 🔻 4. NTLM Authentication is Allowed   
   
  - NTLM is still widely supported for backward compatibility.   
  - It’s vulnerable to **relay attacks** because it doesn’t verify who is asking for authentication.   
   
  > 📌 Kerberos is safer, but NTLM still gets used, especially in fallback scenarios.   

   
### 🔻 5. Target Services Don’t Enforce Signing   
   
  - Services like **LDAP/SMB/HTTP** must explicitly require:   
      - **Message signing**   
      - **Channel binding**   
  - Without these, NTLM can be blindly accepted and relayed.   
   
  > 📌 Relaying only works if the destination doesn’t verify integrity of the connection.   

   
### 🔻 6. Lack of Network Segmentation   
   
  - Attackers can sit on the same subnet and intercept internal traffic.   
  - No isolation = attacker can manipulate service discovery and communication.   
   
  > 📌 Flat networks = paradise for MITM attacks.   

