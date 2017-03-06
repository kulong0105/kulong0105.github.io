---
title: An introduction about cpu cache
category: linux
tags:
- linux 
- cache
---

## 介绍

1. 首先，本文将对3种体系结构的内存访问方式进行介绍，分别为：
   * 对称多处理器结构 (SMP：Symmetric Multi-Processor) 
   * 海量并行处理结构 (MPP：Massive Parallel Processing)
   * 非一致存储访问结构 (NUMA：Non-Uniform Memory Access)

2. 然后，通过3种不同体系结构的内存访问实现原理，来对比它们在性能、扩展以及应用方面的优缺点。

3. 最后，对NUMA结构的内存分配策略以及如何使用numactl工具优化程序进行介绍。

在详细介绍不同体系架构的内存访问方式之前，先对CPU的相关概念、各种总线技术以及内存带宽等概念进行介绍。


完整文章请转到下面link:
[CPU访问内存方式简介](https://pan.baidu.com/s/1dFKe4X7)
