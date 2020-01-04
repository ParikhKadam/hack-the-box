---
title: Hack The Box Sense
published: true
---

Hey Guys This is chan.Today I'll write about sense from hack the box and here is my write up about it.

# [](#header-2)Nmap Scan
```
root@ch4n:~# nmap -sC -sV  10.10.10.60
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-04 20:34 EST
Nmap scan report for 10.10.10.60
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
443/tcp open  ssl/https?

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 121.28 seconds
```
In nmap scan there are two open ports.

port 80 and port 443 So it will be https protoco 

So let's check the webpage 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/webpage.png)

If we look at  the webpage,there is a login page.We get a pfSense login page. pfSense is a free and open-source firewall and router.

So I search in google about default pfsense login creds

```
Username: admin
Password: pfsense
```
but it doesnt work 

# [](#header-3)Gobuster

```
root@ch4n:~/Desktop/htb/boxes/sense# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://10.10.10.60 -k -x php,txt,conf
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.60
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt,conf
[+] Timeout:        10s
===============================================================
2020/01/04 21:18:58 Starting gobuster
===============================================================
/index.php (Status: 200)
/help.php (Status: 200)
/themes (Status: 301)
/stats.php (Status: 200)
/css (Status: 301)
/edit.php (Status: 200)
/includes (Status: 301)
/license.php (Status: 200)
/system.php (Status: 200)
/status.php (Status: 200)
/javascript (Status: 301)
/changelog.txt (Status: 200)
/classes (Status: 301)
/exec.php (Status: 200)
/widgets (Status: 301)
/graph.php (Status: 200)
/tree (Status: 301)
/wizard.php (Status: 200)
/shortcuts (Status: 301)
/pkg.php (Status: 200)
/system-users.txt (Status: 200)
```
So whenI check changelog.txt It said like this 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/change_log.png)
It means that they use the vulnerable version of this software 

so let's check system-users.txt 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/creds.png)

In here we got the user and pass
```
username: rohit
password: company defaults(it is pfsense)defutlt paass
```
So I successfully login with this user and pass

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/login_pannel.png)

After logging in,I noticed that this box using 2.1.3 of pfsense 

So I search for exploit

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/exploit.png)

You can also check [exploit-db](https://www.exploit-db.com/exploits/43560)

# [](#headrer-3)Exploitation
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/using_exploit.png)

![https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/sense/root.jpg]

In here we are with shell as root.So that's it.Pretty easy box.

Thanks for reading.

Feedback is appreciate!

Enjoy:-)
