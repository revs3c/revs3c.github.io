---
layout: post
title:  "HackTheBox-Dog"
type: docs
---

---
Performed Nmap scan → Found Apache with Backdrop CMS.
Discovered .git folder → Dumped source code → Found DB creds.
Used creds to log into admin panel → Uploaded PHP reverse shell using crafted tar.gz.
Gained shell → Escalated to user via reused credentials.
Used sudo bee misconfig to escalate to root.

---

### Nmap Scan

```
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 3836E83A3E835A26D789DDA9E78C5510
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Home | Dog
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
| http-git: 
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Uptime guess: 17.835 days (since Sun Jun 22 18:04:29 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=247 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

In the scan we can see the os is linux and on port 80 is running apache/2.4.41 and hear's a notable thing in the nmap output we can see in the http-title section directing towards a webpage plus a robots.txt file with "/?q=admin" <----- intresting.

At http://10.10.11.58/ we get 

![Image]({{ site.baseurl }}/assets/images/Pastedimage20250710141120.png)


And at http://10.10.11.58/?q=admin, not the access denied, but at the bottom left.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710141232.png)

But no mention of version is there but using out Brahmastra google we get a exploit db page tell a rce on BackDrop CMS. And the login us useless as no defaults are working.

Let's move ahead on directory bruteforce via dirb.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710141923.png)

This in intresting after navigating we can see files of .git folder. "http://10.10.11.58/.git/" lets get it using a tool called git dumper.

```
git-dumper http://10.10.11.58/.git git
```

After downloading those files we get a folder named core/modules and a setting.php file with the password of root.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710142504.png)

And if u are goot at recon u might have noticed the posts in the webpage the user's of dog use email so lets find them.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710142849.png)

ok now that we have got the user email lets try to get in. Using those cred's we get access to admin panel, early i talked about the exploit db exploit.

But the problem hear is that to upload the module it excepts certian format's like 

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710143711.png)

ok so to create one we will need a php shell and a file with .info extension as its a backdrop cms, now the process goes like this.

1. Use the exploit db python script to create a shell.
2. Them replace the code of shell.php with the pentest monkey's php shell.(Change ip and port)
3. the create a tar.gz.
4. and upload the tar.gz to the dog.
5. remember the server delete's the shell immediately so start the listener (``` nc -lvnp 444```)  
6. Last navigate to https://10.10.11.58/modules/shell/shell.php

Doing these steps will give use a reverse shell.

After getting a reverse shell we have the user www-data. when we navigate to /home we get to more users "johncusack" and "" trying the same password as tiffany we get the the user flag.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250712163127.png)

Now escalating more we get by doing ```sudo -l ``` we get the ALL ALL for the bee command.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250712163320.png)

Now hear i don't know much how to escalate by using chatgpt we get to know that bee command is used to manage the backdrop cms.

![image]({{ site.baseurl }}/assets/images/Pastedimage20250710151050.png)

Ok so when we do bee eval to get the root we are greated with error.

Then we get the root.txt

![image]({{ site.baseurl }}/assets/images/Pastedimage20250712165828.png)
