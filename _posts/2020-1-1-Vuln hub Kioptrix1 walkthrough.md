---
title: Vuln hub Kioptrix1 walkthrough
published: true
---
Hey Guys.This is chan and today I'll write about kioptix1 from vuln hub.this is an oscp like box.

So let's get jump in

# [](#header-2)Enumeration
# [](#header-3)Nmap 
```
root@ch4n:~# nmap -sC -sV 192.168.99.227
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-02 15:26 EST
Nmap scan report for 192.168.99.227
Host is up (0.0025s latency).
Not shown: 994 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
|_sshv1: Server supports SSHv1
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: 400 Bad Request
|_ssl-date: 2020-01-03T01:27:41+00:00; +4h59m58s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_DES_64_CBC_WITH_MD5
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:DF:4B:72 (Oracle VirtualBox virtual NIC)

Host script results:
|_clock-skew: 4h59m57s
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 127.27 seconds
```

In our nmap there is  smb "Samba smbd"

So I used metaspoilt to check smb version 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/check%20smb%20version.jpg)

so we got smb version 

Then I search exploit in google for this version 


# [](#header-3)Exploit

https://www.exploit-db.com/exploits/10/

Compiling exploit and excute 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/vuln-hub/root.png)

So here we got root

Enjoy:-)
