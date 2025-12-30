---
layout: defaults
Weight: 2
title:  "LLMNR Attack"
type: docs
---

### What is LLMNR (Link-Local Multicast Name Resolution)?   
   
- LLMNR (Link-Local Multicast Name Resolution) is a fallback mechanism used in Windows when the DNS (Domain Name System) fails to resolve a hostname. It allows devices to resolve hostnames in a local network using multicast queries.   
   
   
### How Does LLMNR Works?   
   
1. **A device needs to resolve a hostname**   
  - When a Windows machine wants to connect to another system (e.g., `fileserver.local`), it first **queries its configured DNS server**.   
  - If DNS **fails** to resolve the hostname, the system falls back to **LLMNR**.   
2. **The system sends an LLMNR multicast query**   
  - The device **broadcasts a request** over UDP port **5355** asking:   
      👉 "*Who has the IP address for fileserver.local?"   
  - This query is sent to **all devices** in the local network.   
3. **A device with that hostname responds**   
  - If a system in the network has the hostname **fileserver.local**, it replies with:   
      👉 "*I am fileserver.local, and my IP is 192.168.1.10."   
  - The requesting machine then **connects to that IP address**.   
4. **If no response, it falls back to NBT-NS**   
  - If LLMNR also **fails**, the system tries **NetBIOS Name Service (NBT-NS) on UDP 137** as a last resort.   
   
### Why LLMNR Is Vulnerable ?   
   
  - **LLMNR (Link-Local Multicast Name Resolution) is a multicast protocol that operates on UDP port 5355.**   
  - **It follows a "First Come, First Serve" principle**, meaning it accepts the first response it receives **without verification**.   
  - **When a system queries "Who has fileserver.local?"**, it blindly trusts the first device that responds.   
  - **If an attacker replies first**, the victim's machine will attempt to connect to the attacker's provided IP, sending its **username and NTLMv2 hash** if authentication is required.   
   
**LLMNR/NBT-NS only works with hostnames**, not direct IP addresses.   
   
```
[SMB] NTLMv2-SSP Client   : fe80::a3f2:5ec8:da22:a818
[SMB] NTLMv2-SSP Username : TONYSTARK\peaterparker
[SMB] NTLMv2-SSP Hash     : peaterparker::TONYSTARK:5dbced5cd8ee4cb3:5B044E2F75A8822373280AE8DC7D0D59:0101000000000000000B4462F9A4DB01CC38E39254DBA78D000000000200080039004E005500490001001E00570049004E002D0051005A00570057004B0038005000350043005700550004003400570049004E002D0051005A00570057004B003800500035004300570055002E0039004E00550049002E004C004F00430041004C000300140039004E00550049002E004C004F00430041004C000500140039004E00550049002E004C004F00430041004C0007000800000B4462F9A4DB010600040002000000080030003000000000000000010000000020000018C6342388B8DEB30F201810A6C776386F1236D60BE01441DB3E6A97F64666330A0010000000000000000000000000000000000009002A0063006900660073002F00660069006C0065007300650072007600650072002E006C006F00630061006C000000000000000000

```
   
