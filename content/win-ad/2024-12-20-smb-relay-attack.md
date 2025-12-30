---
layout: default
Weight: 3
title:  "SMB Relay Attack"
type: docs
---

### What is SMB Relay Attack.   
   
An SMB Relay Attack is a man-in-the-middle (MITM) attack where an attacker captures NTLM authentication requests from a victim and forwards (relays) them to another system (usually a server) to gain access — without ever knowing the user’s password.   
   
### How Does SMB Relay Attack Works ?!   
   
#### Step 1: Set up Responder (LLMNR/NBT-NS Poisoning)   
   
```
sudo responder -I eth0 # With no smb server running

```
   
#### Step 2: Set up ntlmrelayx to Relay the Credentials   
   
```
sudo ntlmrelayx.py -t smb://<target-ip> -smb2support -i #This will relay the hashe's
```
   
> -t smb://<target-ip>: Relay the NTLM credentials to this machine.   

> -smb2support: Support for SMBv2 targets.   

> -i: Interactive shell if successful.   

   
#### Step 3: Wait for Victim to Authenticate   
   
Now, when the victim tries to access a non-existent share (like `\\fake-server\share`), their system sends an LLMNR/NBT-NS broadcast.   
   
- Responder replies: "I’m `fake-server`!"   
- Victim sends NTLM authentication info.   
- ntlmrelayx receives it and **relays** it to your target server.   
- If successful, you get:   
    A shell prompt or hash dump like:   
```
[+] Dumping SAM hashes
frank:500:aad3b435b51404eeaad3b435b51404ee:<NTLM hash>:...

```
   
   
### Why It's Vulnerable ?!   
   
NTLM **does not bind** the authentication to the source IP, hostname, or context of the original client. That’s the core of the vulnerability.  
 
So, when `ntlmrelayx` captures a valid NTLM authentication attempt from the victim and then **relays** it to another machine (the target), the target sees it as a **legitimate authentication request** — it only sees that:   
  - A user (e.g., `DOMAIN\victimuser`)   
  - Is trying to authenticate using NTLM   
  - With a valid challenge/response (the hash)   
### 🛡️ Unless SMB signing or Extended Protection for Authentication (EPA) is enabled:   
   
  - The target **has no way** to know this NTLM request was **relayed** from an attacker.   
  - There's **no verification** that it originated from the actual client.   
###  Summary:   
   
  - ✅ `ntlmrelayx` can relay NTLM authentication to any server that **accepts NTLM** and has **signing disabled**.   
  - ❌ The target does **not check the origin** of the request.   
  - 🧠 That’s why NTLM relay is so dangerous — it’s a d**esign flaw **in how NTLM works.
   
   
     
   
