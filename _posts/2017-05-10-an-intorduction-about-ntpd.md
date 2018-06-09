---
title: NTP简介
category: linux
tags:
- linux
---

## 介绍

NTP（Network Time Protocal, 网络时间协议）是使网络中的各个计算机时间同步的一种协议，
本文将对ntp的基本概念和Server/Client配置进行介绍。


## 基本概念

### 硬件时间与系统时间

- Real Time Clock(RTC), 也即硬件时钟时间，是指嵌在主板上的特殊的电路, 在系统关机时，时间仍继续计算
- System Clock, 操作系统系统，它指从1970年1月1日00:00:00 UTC时间到目前为止秒数总和的值, 系统时间在开机的时候会和硬件时间同步
- 硬件时间和系统系统的查看和设置

```
# date                                       # 显示系统时间
# hwclock -r                                 # 显示硬件时间
# date -s "date -s "2017/5/9 10:54:00"       # 设置系统时间为指定时间
# hwclock --set --date "2017/5/9 10:54:00"   # 设置硬件时间为指定时间
# hwclock -w         # 设置硬件时间为系统时间
# hwclock -s         # 设置系统时间为硬件时间
```

### ntpd与ntpdate

- ntpdate和ntpd是互斥的，两者不能同时使用
- ntpd是步进式平滑的逐渐调整时间, ntpdate不会考虑其他程序是否会阵痛，直接调整时间

<!--more-->


## 准备工作

- CentOS 7系下会默认安装chronyd, ntp与其互斥，需要将其卸载或关闭其服务
- 为了统一时区, 在服务器上执行`ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime`


## ntpd配置文件

ntpd的配置文件为/etc/sysconfig/ntpd，默认配置如下：
```
# cat /etc/sysconfig/ntpd
# Command line options for ntpd
OPTIONS="-g"
```

说明：
- `man ntpd`可以看出，默认情况下，首次只可以同步时,server/client间时差不能超过1000s，使用这个参数，首次同步不受任何限制
- 默认情况下，ntpd只会通过系统时间，如果也需要通过硬件时间，可以添加`SYNC_HWCLOCK=yes`到配置文件中


## ntpd配置参数

```
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

driftfile /var/lib/ntp/drift
# 计算本ntp server 与上层ntpserver的频率误差

# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
#restrict default nomodify notrap nopeer noquery
restrict default nomodify

# restrict可以限制客户端权限，可以使用的 parameter说明：
# kod            kod技术可以阻止“Kiss of Death “包对服务器的破坏
# nomodity       client可通过ntp进行时间同步，但不能通过 ntpq, ntpc 等更改server参数
# notrap         不提供trap远程登陆 (remote event logging) 功能
# nopeer         不与其它同一层的ntp server进行时间同步
# noquery        客户端不能够使用 ntpq, ntpc 等指令来查询时间服务器，即拒绝ntp时间同步；
# notrust        拒绝无认证的client
# ignore         拒绝所有连接到ntp server的请求

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1  ##打开允许本地所有操作 
restrict ::1
# Hosts on local network are less restricted.
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

# restrict 用来分配指定网段权限，格式如下：
# restrict ［授权同步的网段］ mask ［netmask］ ［parameter］
# 例：restrict 172.16.1.0 mask 255.255.252.0 nomodify
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server time1.aliyun.com
server time2.aliyun.com
server time3.aliyun.com
server time4.aliyun.com
server time5.aliyun.com prefer
#server 用来设置上一级ntp服务器，这里使用阿里云提供的ntp服务器，parameter说明:
# prefer     最高优先级
# burst      当一个运程NTP服务器可用时，向它发送一系列的并发包进行检测。
# iburst     当一个运程NTP服务器不可用时，向它发送一系列的并发包进行检测。

# 如果无法与上层ntp server通信以本地时间为标准时间
server 127.127.1.0 # local clock
fudge 127.127.1.0 stratum 10
#broadcast 192.168.1.255 autokey	# broadcast server
#broadcastclient			# broadcast client
#broadcast 224.0.1.1 autokey		# multicast server
#multicastclient 224.0.1.1		# multicast client
#manycastserver 239.255.254.254		# manycast server
#manycastclient 239.255.254.254 autokey # manycast client

# Enable public key cryptography.
#crypto

includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating
# with symmetric key cryptography. 
keys /etc/ntp/keys

# Specify the key identifiers which are trusted.
#trustedkey 4 8 42

# Specify the key identifier to use with the ntpdc utility.
#requestkey 8

# Specify the key identifier to use with the ntpq utility.
#controlkey 8

# Enable writing of statistics records.
#statistics clockstats cryptostats loopstats peerstats

# Disable the monitoring facility to prevent amplification attacks using ntpdc
# monlist command when default restrict does not include the noquery flag. See
# CVE-2013-5211 for more details.
# Note: Monitoring will not be disabled with the limited restriction flag.
disable monitor```

```


## ntpd状态检查

### ntpq

```
# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 localhost       .STEP.          16 l    - 1024    0    0.000    0.000   0.000
*app-0.localhost 120.25.115.20    3 u  570 1024  377    0.113    0.522   0.360
[root@skyaxe-computing-0 ~]#
```

- remote: 它指的就是本地机器所连接的远程NTP服务器
- refid: 它指的是给远程服务器提供时间同步的服务器
- st: 远程服务器的层级别（stratum), 0 for local reference clocks, 1 for servers with local reference clocks, ..., 16 for unsynchronized server clocks
- t:  u: unicast or manycast client, b: broadcast or multicast client, p: pool source, l: local (reference clock), s: symmetric (peer), A: manycast server, B: broadcast server, M: mul‐ticast server
- when: 自从上次接受到信息到现在已经过去多久了，‘-’表示从没有接受到任何数据信息
- poll: 本地机和远程服务器多少时间进行一次同步(单位为秒)
- reach: 一个八进制值,用来测试能否和服务器连接.每成功连接一次它的值就会增加
- delay: 从本地机发送同步要求到服务器的往返时间
- offset: 这是个最关键的值, 它告诉了我们本地机和服务器之间的时间差别. offset越接近于0,我们就和服务器的时间越接近
- jitter: 这是一个用来做统计的值. 它统计了在特定个连续的连接数里offset的分布情况. 简单地说这个数值的绝对值越小我们和服务器的时间就越精确

需要注意的是，app-0.localhost前面有个*号，其实总共有4中可能的符号: "*, +, -, x "
- 星号：远端的服务器已经被确认为主NTP Server, 本机的系统的时间将由这台机器所提供
- 加号：作为辅助的NTP Server和带有*号的服务器一起为我们提供同步服务,当*号服务器不可用时它就可以接管
- 减号：远程的服务器被认为是不合格的NTP Server
- 乘号：远程的服务器不可用


### ntpstat

```
# ntpstat
synchronised to NTP server (10.0.0.10) at stratum 4
   time correct to within 83 ms
   polling server every 1024 s
#
```


## 实际配置

实际使用的测试配置文件：

### server端配置

```
$ cat /etc/ntp.conf

driftfile /var/lib/ntp/drift

# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default nomodify notrap nopeer noquery

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1
restrict ::1

# Hosts on local network are less restricted.
restrict 192.168.0.0 mask 255.255.0.0 nomodify   # 客户端的子网

server 1.cn.pool.ntp.org
server 0.asia.pool.ntp.org
server 2.asia.pool.ntp.org

server 127.127.1.0
fudge 127.127.1.0 stratum 10

includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

### client端配置

```
[renyl@localhost config]$ cat client.conf

driftfile /var/lib/ntp/drift

# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default nomodify notrap nopeer noquery

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1
restrict ::1

server 192.168.50.3   # ntpd服务器的IP地址

includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```


## 参考
- [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s1-ntp_strata](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s1-ntp_strata)
- [https://blog.csdn.net/iloli/article/details/6431757](https://blog.csdn.net/iloli/article/details/6431757)
- [https://blog.coor.fun/?p=113](https://blog.coor.fun/?p=113)
