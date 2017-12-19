---
title: systemd简介
category: linux
tags:
- systemd
---

## 介绍

本文主要进行如下介绍：

* systemd背景
* systemctl用法
* 运行级别说明


### systemd背景

* systemd 是 Linux 下一个与 SysV 和 LSB 初始化脚本兼容的系统和服务管理器。
* systemd 无需经过任何修改便可以替代 sysvinit 
* systemd 的世界里 service 和 chkconfig 命令将继续像之前一样工作。
* systemd 开启和监督整个系统是基于 unit 的概念。

<!--more-->

### systemctl用法

#### Unit状态

| 序号 | 命令| 说明| 
| --- | --- | --- |
| 1 | systemctl | 列出正在运行的进程信息| 
| 2 | systemd-cgls | 列出正在运行的进程信息| 
| 3 | systemctl -t service | 列出所有正在运行的服务 | 
| 4 | systemctl is-active xxx.service | 服务是否正在运行 | 
| 5 | systemctl is-failed xxx.service | 服务是否启动失败 | 
| 6 | systemctl status xxx.service | 服务当前状态详细信息 | 
| 7 | systemctl is-enabled xxx.service | 服务是否开启自启动 |


#### Unit管理

| 序号 | 命令| 说明| 
| --- | --- | --- |
| 1 | systemcal daemon-reload | 重载所有修改过的配置文件 | 
| 2 | systemctl reload xxx.service |重载服务配置文件| 
| 3 | systemctl start xxx.service | 启动服务| 
| 4 | systemctl stop xxx.service | 停止服务| 
| 5 | systemctl kill xxx.service | kill服务| 
| 6 | systemctl restart xxx.service | 重启服务| 
| 7 | systemctl enable xxx.service | 设置开机子启动|  
| 8 | systemctl disable xxx.service | 取消开机自启动|


#### 其他

| 序号 | 命令| 说明| 
| --- | --- | --- |
| 1 | systemctl list-units | 列出正在运行的 Unit | 
| 2 | systemctl list-unit-files | 列出所有配置文件|
| 3 | systemctl list-dependencies xxx.service | 列出服务的依赖|
| 4 | systemd-analyze | 查看启动耗时 |
| 5 | systemd-analyze blame | 查看每个服务启动耗时 |
| 6 | systemd-analyze critical-chain | 显示瀑布状的启动过程流 |
| 7 | systemctl get-default | 查看启动时的默认 Target |
| 8 | systemctl set-default multi-user.target | 设置启动时的默认 Target | 

### 运行级别说明

| Sysvinit运行级别 | Systemd目标| 说明| 
| --- | --- | --- |
| 0 | runlevel0.target, poweroff.target | 关闭系统 |
| 1 | runlevel1.target, rescue.target | 单用户模式 |
| 2,4 | runlevel2.target, runlevel4.target, multi-user.target | 用户定义/域特定运行级别默认等同于 3 |
| 3 | runlevel3.target, multi-user.target | 多用户非图形化，通过多个控制台或网络录。 |
| 5 | runlevel5.target, graphical.target | 通常为所有运行级别3的服务外加图形化录 |
| 6 | runlevel6.target, reboot.target | 重启 |
| emergency | emergency.target | 紧急 Shell |


### 改变运行级别

改变至多用户（非图形化）运行级别：
* 6系：telinit  3

* 7系
	** systemctl isolate multi-user.target 
	** systemctl isolate runlevel3.target 
	** telinit 3

设置在下一次启动时使用多用户（非图形化）运行级别：
* 6系：修改/etc/inittab
* 7系：ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
 
关机：
poweroff | halt -p | init 0 | shutdown -P now

