---
title: SSH实现原理及实践
category: linux
tags:
- ssh
---

## 概述

SSH全称Secure shell，顾名思义其意思就是非常安全的shell。SSH的主要目的是用来取代传统的telnet和R系列命令（rlogin,rsh,rexec等），通过加密的方式来远程登录和远程执行命令，从而防止由于网络监听而出现的密码泄漏等情况。

SSH是一种加密协议，不仅在登陆过程中对密码进行加密传送，而且对登陆后执行的命令的数据也进行加密，这样即使别人在网络上监听并截获了你的数据包，也看不到其中的内容。
   
<!--more--> 

完整文章请转到下面link:
[SSH实现原理及实践](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/ssh%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E8%B7%B5.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/ssh%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E8%B7%B5.pdf)
