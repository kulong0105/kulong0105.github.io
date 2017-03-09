---
title: LVM Snapshot简介
category: linux
tags:
- lvm
- snapshot
---

## 介绍

1. Logical Volume Manager (LVM)提供了对任意一个Logical Volume(LV)做“快照”(snapshot)的功能，
   以此来获得一个分区一致性备份。

2. LVM的snapshot是通过“写时复制”(copy on write) 的方法实现：
   *  当一个snapshot创建的时候，仅拷贝原始卷里数据的元数据(meta-data)。创建的时候，
      并不会有数据的物理拷贝，因此snapshot的创建几乎是实时的。
   *  当原始卷上有写操作执行时，snapshot跟踪原始卷块的改变，这个时候原始卷上将要改
      变的数据在改变之前被拷贝到snapshot预留的空间里，因此这个原理的实现叫做写时复制(copy-on-write)。

3. 在写操作写入块之前，CoW会将原始数据移动到snapshot空间里，这样就保证了所有的数据在
   snapshot创建时保持一致。而对于snapshot的读操作，如果是读取数据块是没有修改过的，
   那么会将读操作直接重定向到原始卷上，如果是要读取已经修改过的块，那么就读取拷贝到snapshot中的块。

<!--more-->

完整文章请转到下面link:
[LVM Snapshot简介](https://pan.baidu.com/s/1dFKe4X7)
