---
layout: default
title:  "Abusing Login's"
weight: 3
type: docs
---

## Startup Folders

The startup folders are those folders which run the filles or rather which runs any file present in it specifically speaking. which runs at user logins aka "start up processes" there are two types of those folders.

- User specific present at<br>
 ```C:\Users\<your_username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup```
</br>
- For all users
  ```C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp```

Hear droping any executable file will lead to a shell.

## Run/RunOnce Via Registry

In this method insted of placing executable in startup folders, hear we direct to our executable via registry which can be ran as we want.

The following registrys
- ```HKCU\Software\Microsoft\Windows\CurrentVersion\Run```
- ```HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce```
- ```HKLM\Software\Microsoft\Windows\CurrentVersion\Run```
- ```HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce```

These are the one where we can edit and direct to our executable the one's with **HKCU** are user specific and **HKLM** are for all users.

Placing our executable in **RUN** will execute it every time a user logs in and plaing our executable in **RUNOnce** will lead to one time execution.

![image](../../images/R1.png)

## WinLogon Registry
Another simple yet user full startup persistance is via WINlogon registrys as it loads the user profile first among any other things.

The WINlogon registrys are present under ```HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\``` as its consist of 
- ```Userinit``` points to ```userinit.exe```, which is in charge of restoring your user profile preferences.
- ```shell``` points to the system's shell, which is usually ```explorer.exe```.

![image](../../images/R2.png)

Hear we can do 2 things either we can place a executable like shown above and it can also process caommands which can be seperated by a comma.

But specifically in **userinit** field we need to keep the **userinit.exe** file as it might break the user sequence.But as said earlyer we can execute commands by commas.

![image](../../images/R3.png)

## Logon scripts 
This method is user specific only where we will be abusing the environment variable called ```UserInitMprLogonScript```. Now this is executed when the ```userinit.exe``` is invoked at the user login session as seen in abover method.

This variable can be found in ```HKCU\Environment``` note hear its ```HKUC``` as said this is user specific.

{{< callout type="info" >}}
  The variable isn't set by default, so we can just create it and assign any script we like.
{{< /callout >}}

![image](../../images/R4.png)

