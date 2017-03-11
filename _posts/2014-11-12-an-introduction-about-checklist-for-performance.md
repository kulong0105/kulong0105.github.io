---
title: 性能检证Checklist
category: performance
tags:
- checklist
---

本文归纳OS性能检证中的Checklist，包括：
- OS 版本
- Kernel 版本
- 常用服务
- CPU
- Memroy
- Disk
- Network

<!--more-->

## 1. 系统版本

* release版本

| 适合范围 | 确认方法| 
| --- | --- | 
| Native、LXC、Guest | cat /etc/redhat-release | 

* kernel版本

| 适合范围 | 确认方法| 
| --- | --- | 
| Native、LXC、Guest | uname -r | 


## 2. 常用服务

* selinux

| 适合范围 | 确认方法| 修改方法 |
| --- | --- | --- |
| Native、Guest | getenforce | 修改/etc/selinux/config文件，需重启生效 |


* audit

| 适合范围 | 确认方法| 修改方法 |
| --- | --- | --- |
| Native、Guest | service auditd status | service auditd start/stop/restart |

注：
1）7系下重启后永久生效：systemctl  disable/enable  auditd.service
2）6系下重启后永久生效：chkconfig  --level 345 auditd off/on 

* ftrace

| 适合范围 | 确认方法| 修改方法 |
| --- | --- | --- |
| Native、Guest | /sys/kernel/debug/tracing/tracing_on | 设置为0或1 |

注：当tracing_on为1，current_tracer为非nop时，trace表示打开。

* tuned

| 适合范围 | 确认方法| 修改方法 |
| --- | --- | --- |
| Native | service tuned status | service tuned stop/start/restart |

注:
profile模式：tuned-adm list
tuned-adm profile xxx


## 3. CPU

| 适合范围 | 确认方法| 
| --- | --- | 
| Native | 查看/proc/cmdline是否有isolcpus、intel_idle.max_cstate选项, /sys/devices/system/cpu/cpuN/online,cpu是否被禁用 | 
| Guest  | 查看虚拟机的.xml文件中vcpu num、vcpupin、emulatorpin的参数, 虚拟机的CPU特性（即Copy Host mode）|


## 4. Memory

| 适合范围 | 确认方法| 
| --- | --- | 
| Native | 查看/proc/cmdline是否有Mem选项（对内存大小进行限制）|
| Guest  |  查看虚拟机的.xml文件中memory参数（内存大小的设定） |

## 5. Disk

| 适合范围 | 确认方法| 
| --- | --- | 
| Native | 查看磁盘的文件系统、大小： df –hT, 磁盘的cache模式： dmesg | grep sd* , scheduler：cat /sys/block/sd*/queue/scheduler |
| Guest  | 查看虚拟磁盘的Disk bus、Strorage format、cache mode、io mode  参数值（一般设置为virtio、raw、none、native）和scheduler| 

## 6. Network

| 适合范围 | 确认方法| 
| --- | --- | 
| Native | 查看是否设置了bridge： brctl show , 网卡信息： ethtool  eth0 | 
| Guest  | 查看虚拟机网络的Source device及Device model |

## 7. 其他

* Cgroup的设定

| 适合范围 | 确认方法| 
| --- | --- | 
| Native、LXC、Guest| cpu相关信息：/sys/fs/cgroup/cpuset/ 目录下查看 , 内存相关信息：/sys/fs/cgroup/memory/目录下查看 |

* 7.2 getPerinfo.sh的安装

| 适合范围 | 确认方法| 
| --- | --- | 
| Native、LXC、Guest| which getPerfinfo.sh |

* 7.3 virt-top的安装

| 适合范围 | 确认方法| 
| --- | --- | 
| Host | which virt-top |

* 7.4 kvm_stat的安装

| 适合范围 | 确认方法| 
| --- | --- | 
| Host | which kvm_stat |


