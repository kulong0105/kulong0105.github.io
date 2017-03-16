---
title: 窥探CPU Cache
category: linux
tags:
- linux 
- cache
---

## 介绍

1. CPU负责程序指令的执行，内存负责数据的存储，但CPU的执行速度远大于内存的访问速度，
为了缓和两者之间的速度差异，于是CPU Cache就应运而生了。

2. CPU Cache由硬件实现，速度介于寄存器和内存之间，系统会把经常使用的数据放到Cache中，
当对相同数据进行多次操作时，就可以避免从内存中获取数据，而直接从CPU Cache中获取数据，这样就会提高程序性能。

3. CPU Cache 在高级语言（C，C++，Java等）程序员的角度来看，它是透明的，无法直接干预它，
也无法察觉它是如何运行的，因为它是完全依赖硬件设施来实现的。但是，我们可以利用它，
在并发编程中善于运用CPU Cache，会给程序性能带来“质”的提升。

本文主要通过3个知识点入口，每个知识点将结合一个程序来说明CPU Cache的工作原理及相关技术。

<!--more-->

完整文章请转到下面link:
[窥探CPU Cache](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/%E7%AA%A5%E6%8E%A2CPU%20Cache.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/%E7%AA%A5%E6%8E%A2CPU%20Cache.pdf)
