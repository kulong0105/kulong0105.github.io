---
title: mlock实现原理及应用
category: linux
tags:
- linux 
- mlock
---

## mlock简介

*  mlock（memory locking）是内核实现锁定内存的一种机制，用来将进程使用的部分或全部虚拟内存锁定到物理内存。
*  mlock机制主要有以下功能：
   - 被锁定的物理内存在被解锁或进程退出前，不会被页回收流程处理。
   - 被锁定的物理内存，不会被交换到swap设备。
   - 进程执行mlock操作时，内核会立刻分配物理内存。
   
<!--more--> 

完整文章请转到下面link:
[mlock实现原理及应用](https://pan.baidu.com/s/1dFKe4X7)
