---
title: Linux内核同步机制简介
category: linux
tags:
- sync
---

## 介绍

1. 由于现代Linux操作系统是多任务、SMP、抢占式以及中断是异步执行的，导致共享资源容易
   被并发访问，从而使得访问共享资源的各线程之间互相覆盖共享数据，造成被访问数据处于
   不一致状态，因此Linux提供了同步机制来防止并发访问。

2. 常用的同步机制（如自旋锁）用来保护共享数据使用起来简单有效，但由于CPU的处理速度与
   访问内存的速度差距越来越大，导致获取锁的开销相对于CPU的速度在不断的增加。因为这种
   锁使用了原子操作指令，需要原子地访问内存，即获取锁的开销与访问内存的速度相关。

3. Linux内核根据对不同共享资源的特性，提供多种同步机制：原子操作、自旋锁、读-写自旋锁、
   信号量、读-写信号量、完成变量、顺序锁、禁止抢占、内存屏障及RCU，本文将对其分别进行简要介绍。

<!--more-->

完整文章请转到下面link:
[Linux内核同步机制简介](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/Linux%E5%86%85%E6%A0%B8%E5%90%8C%E6%AD%A5%E6%9C%BA%E5%88%B6%E7%AE%80%E4%BB%8B.docx)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/Linux%E5%86%85%E6%A0%B8%E5%90%8C%E6%AD%A5%E6%9C%BA%E5%88%B6%E7%AE%80%E4%BB%8B.docx)
