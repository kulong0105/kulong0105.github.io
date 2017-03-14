---
title: Linux tuned服务简介
category: linux
tags:
- tuned
---

## 介绍

本文将对tuned服务做简要介绍, 主要内容为：

* tuned服务的作用
* tuned服务的调优

### 1. tuned服务的作用

通过udev来监视硬件设备，然后根据监视所获取的数据来对系统进行动态调优或直接静态调优。

### 2. tuned服务的两类调优程序											

* 监视程序：主要负责对系统的硬件设备进行监视，详细信息如下：

| 序号 | 监视范围 |	描述 |
| --- | --- | --- |
| 1	| disk | 每间隔一定时间获取系统每个磁盘的负载（IO操作的数量）|
| 2	| net  | 每间隔一定时间获取系统每个网卡的网络负载（传输数据包的数量）|
| 3	| load | 每间隔一定时间获取系统每个CPU的负载（运行时间）|

注：默认的间隔时间为10秒，可以通过文件/etc/tuned/tuned-main.conf里的参数update_interval来调整。

<!--more-->
																			
完整文章请转到下面link:
[Linux tuned服务简介](https://pan.baidu.com/s/1dFKe4X7)
