---
title: Hack The Box Inferno Write Up
published: true
---

Hey Guys This is chan and today I'll write about inferno challnege from hack the box.It was retired and it is an easy misc challenge.

# [](#header-3)Challenge description

Find the flag

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20challenge/inferno/challenge.jpg)

First it gives us base64 text then I decode it 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20challenge/inferno/text.jpg)

```
RCdgXyReIjdtNVgzMlZ4ZnZ1PzFOTXBMbWwkakdGZ2dVZFNiYn08eyldeHFwdW5tM3Fwb2htZmUrTGJnZl9eXSNhYFleV1Z6PTxYV1ZPTnJMUUpJTkdrRWlJSEcpP2MmQkE6Pz49PDVZenk3NjU0MzIrTy8uJyYlJEgoIWclJCN6QH59dnU7c3JxdnVuNFVxamlubWxlK2NLYWZfZF0jW2BfWHxcW1pZWFdWVVRTUlFQMk5NRktKQ0JmRkU+JjxgQDkhPTw1WTl5NzY1NC0sUDAvby0sJUkpaWh+fSR7QSFhfXZ7dDpbWnZvNXNyVFNvbm1mLGppS2dgX2RjXCJgQlhXVnpaPDtXVlVUTXFRUDJOR0ZFaUlIR0Y/PmJCJEA5XT08OzQzODFVdnUtMiswLygnSysqKSgnfmZ8Qi8=
```

```
D'`_$^"7m5X32Vxfvu?1NMpLml$jGFggUdSbb}<{)]xqpunm3qpohmfe+Lbgf_^]#a`Y^WVz=<XWVONrLQJINGkEiIHG)?c&BA:?>=<5Yzy765432+O/.'&%$H(!g%$#z@~}vu;srqvun4Uqjinmle+cKaf_d]#[`_X|\[ZYXWVUTSRQP2NMFKJCBfFE>&<`@9!=<5Y9y7654-,P0/o-,%I)ih~}${A!a}v{t:[Zvo5srTSonmf,jiKg`_dc\"`BXWVzZ<;WVUTMqQP2NGFEiIHGF?>bB$@9]=<;4381Uvu-2+0/('K+*)('~f|B/
```

We got nonsense text.Honestly, I solved it like this challnege in one of ctf competation

It is an estoteric language call malbolge.You can read about it [here](https://en.wikipedia.org/wiki/Malbolge)

I decode at this [http://malbolge.doleczek.pl/](http://malbolge.doleczek.pl/)

Then Grab the flag 

![](https://raw.githubusercontent.com/Cnw311/hack-the-box/gh-pages/assets/Hack%20the%20box%20challenge/inferno/flag.png)

That's it Guys

Enjoy my write up:-)
