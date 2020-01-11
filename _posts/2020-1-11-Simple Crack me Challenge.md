---
title: Simple Crack me Challenge 1(keyg3nme)
published: true
---

Hey Guys This is the challenge called keyg3nme write up from crackmes.one

This is an very easy reverse enginnering for beginner who wanna start re.

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/challenge.png)

you can download from [here](https://crackmes.one/static/crackme/5da31ebc33c5d46f00e2c661.zip)

frist we extract the zip file (password - crackmes.one)

I check the file with file cmd to know what is the type of the file

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/file.jpg)
So it is ELF 64-bit excutable file type 

Running Birnary file

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/running-binary-file.jpg)

the program ask us to enter the key so I type name and it said nope because we dont have correct key 

So let's open our binary file with ghidra 

When I chcek the main function,it looks interesting 
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/ghidra.png)

```
undefined8 main(void)

{
  int iVar1;
  long in_FS_OFFSET;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Enter your key:  ");
  __isoc99_scanf(&DAT_0010201a,&local_14);
  iVar1 = validate_key((ulong)local_14);
  if (iVar1 == 1) {
    puts("Good job mate, now go keygen me.");
  }
  else {
    puts("nope.");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
So when we check the code validate_key function is interesting one 

iVar1 is the variable that will return as 1.

local_14 is getting passed in a function named validate_key, the returned result will be saved in ivar1


Let's check the validate_key function 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/validate-key.png)

```
ulong validate_key(int iParm1)

{
  return (ulong)(iParm1 % 0x4c7 == 0);
}
```

iParm1 is our entered key 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/0x4c17.jpg)

0x4c7 is 1223

it is equal to 0 

So if we put our enter key as 1223 we will get the keygen 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/final1.jpg)

we can also put 0 as our enter key cause 0x4c7 == 0
![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/cracmes/keyg3nme/final2.jpg)

That's it 

sorry for my bad explantation

I hope you will understand my write up

Enjoy :-)
