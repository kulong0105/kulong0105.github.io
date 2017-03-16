---
title: CPU访问内存方式简介
category: linux
tags:
- linux 
- cache
---

## 介绍

1. 首先，本文将对3种体系结构的内存访问方式进行介绍，分别为：
   - 对称多处理器结构 (SMP：Symmetric Multi-Processor) 
   - 海量并行处理结构 (MPP：Massive Parallel Processing)
   - 非一致存储访问结构 (NUMA：Non-Uniform Memory Access)

2. 然后，通过3种不同体系结构的内存访问实现原理，来对比它们在性能、扩展以及应用方面的优缺点。

3. 最后，对NUMA结构的内存分配策略以及如何使用numactl工具优化程序进行介绍。

在详细介绍不同体系架构的内存访问方式之前，先对CPU的相关概念、各种总线技术以及内存带宽等概念进行介绍。

<!--more-->

完整文章请转到下面link:
[CPU访问内存方式简介](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/CPU%E8%AE%BF%E9%97%AE%E5%86%85%E5%AD%98%E6%96%B9%E5%BC%8F%E7%AE%80%E4%BB%8B.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/CPU%E8%AE%BF%E9%97%AE%E5%86%85%E5%AD%98%E6%96%B9%E5%BC%8F%E7%AE%80%E4%BB%8B.pdf)
