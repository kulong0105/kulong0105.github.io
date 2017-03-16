---
title: 深入理解Glibc的内存管理
category: linux
tags:
- glibc
---

## 介绍

Linux操作系统中的内存管理分为两个部分：
* 物理内存的管理：该部分由Linux内核的buddy系统和slab分配器合作管理，
  管理着物理内存资源的分配及回收。

* 虚拟内存的管理：该部分由Glibc进行管理，给用户层提供了多个关于内存管理方面的函数，
  其中malloc()和free()最为常用。

本文主要讨论Glibc对虚拟内存的管理，分为如下几个部分：
* 首先，通过对Glibc的内存管理机制进行研究，认识到某些情况下不适合用Glibc来管理内存，
  因为其设计的管理机制在某些情况下可能导致内存泄露、大量的内存碎片、内存暴增等问题。

* 然后，通过一些案例来说Glibc管理机制的特性以及如何运用适当的方法来规避Glibc在某些
  情况下对内存管理的缺陷。

<!--more-->

完整文章请转到下面link:
[深入理解Glibc的内存管理](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glibc%E7%9A%84%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Glibc%E7%9A%84%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86.pdf)
