---
title: 深入理解irqbalance
category: linux
tags:
- irqbalance
---

## 介绍

1. irqbalance 服务用于自动优化硬件中断分配。它通过对定期采集（每隔10秒采集一次）
的系统数据（CPU softirq和irq负载）进行分析，根据分析的结果来修改文件（/proc/irq/N/smp_affinity）
的值（即改变中断号的亲属性），从而达到优化硬件中断分配的目的。

2. irqbalance分为两种模式：Performance mode（默认模式）和 Power-save mode。
   1) Performance mode时，irqbalance会将中断尽可能均匀地分配给每个CPU，从而充分利用CPU多核特性来提升性能。
   2) Power-save mode时，irqbalance会将中断集中分配给某个CPU，从而保证其它空闲 CPU的睡眠时间，降低系统能耗。


<!--more-->

完整文章请转到下面link:
[深入理解irqbalance](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3irqbalance.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3irqbalance.pdf)
