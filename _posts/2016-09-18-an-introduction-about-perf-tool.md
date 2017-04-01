---
title: 性能分析工具简介
category: linux
tags:
- linux
- performance
---

## 介绍

本文主要对Linux常用的性能分析工具进行介绍.

## 性能分析工具

### top

动态地显示系统的整体运行状况

```
[renyl@localhost ~]$ top -b -d 1 -n 1 | head -10
top - 22:57:49 up 6 min,  1 user,  load average: 0.00, 0.05, 0.03
Tasks: 241 total,   1 running, 240 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.8 us,  0.9 sy,  0.1 ni, 96.7 id,  0.4 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  7868508 total,  6128392 free,   809416 used,   930700 buff/cache
KiB Swap:  1952764 total,  1952764 free,        0 used.  6628776 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1719 renyl     20   0 2037956 143124  60636 S  12.5  1.8   0:06.05 gnome-shell
 2242 renyl     20   0  758876  36848  27336 S   6.2  0.5   0:02.29 gnome-terminal-
    1 root      20   0  128304   5948   3948 S   0.0  0.1   0:01.58 systemd
[renyl@localhost ~]$
```

<!--more-->

### uptime

显示过去1分钟，5分钟，15分钟的系统平均负载

```
[renyl@localhost ~]$ uptime
22:56:57 up 5 min,  1 user,  load average: 0.00, 0.06, 0.03
[renyl@localhost ~]$
```
注：在4个CPU的系统上，负载为1，表示75%的时间是空闲的。


### ps

显示系统进程相关信息

```
[renyl@localhost ~]$ ps faux | head -5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         2  0.0  0.0      0     0 ?        S    22:51   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    22:51   0:00  \_ [ksoftirqd/0]
root         4  0.1  0.0      0     0 ?        S    22:51   0:00  \_ [kworker/0:0]
root         5  0.0  0.0      0     0 ?        S<   22:51   0:00  \_ [kworker/0:0H]
[renyl@localhost ~]$ ps -ef | head -5
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 22:51 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2     0  0 22:51 ?        00:00:00 [kthreadd]
root         3     2  0 22:51 ?        00:00:00 [ksoftirqd/0]
root         4     2  0 22:51 ?        00:00:00 [kworker/0:0]
[renyl@localhost ~]$ ps -eLf | head -5
UID        PID  PPID   LWP  C NLWP STIME TTY          TIME CMD
root         1     0     1  0    1 22:51 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2     0     2  0    1 22:51 ?        00:00:00 [kthreadd]
root         3     2     3  0    1 22:51 ?        00:00:00 [ksoftirqd/0]
root         4     2     4  0    1 22:51 ?        00:00:00 [kworker/0:0]
[renyl@localhost ~]$ ps axo user,pid,priority,nice,command | head -5
USER       PID PRI  NI COMMAND
root         1  20   0 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  20   0 [kthreadd]
root         3  20   0 [ksoftirqd/0]
root         5   0 -20 [kworker/0:0H]
[renyl@localhost ~]$ ps -C chrome | head -5
  PID TTY          TIME CMD
 4839 pts/4    00:00:06 chrome
 4848 pts/4    00:00:00 chrome
 4853 pts/4    00:00:00 chrome
 4942 pts/4    00:00:03 chrome
[renyl@localhost ~]$
```


### pstree

以树状形式显示系统进程相关信息

```
[renyl@localhost ~]$ pstree -p | head -10
systemd(1)-+-ModemManager(742)-+-{gdbus}(779)
           |                   `-{gmain}(768)
           |-NetworkManager(816)-+-{gdbus}(851)
           |                     `-{gmain}(848)
           |-abrt-dump-journ(881)
           |-abrt-dump-journ(883)
           |-abrtd(817)-+-{gdbus}(880)
           |            `-{gmain}(878)
           |-accounts-daemon(754)-+-{gdbus}(778)
           |                      `-{gmain}(766)
[renyl@localhost ~]$
```


### free

系统系统内存相关信息

```
[renyl@localhost ~]$ free -mh
              total        used        free      shared  buff/cache   available
Mem:           7.5G        813M        5.8G        141M        945M        6.3G
Swap:          1.9G          0B        1.9G
[renyl@localhost ~]$
```


### mpstat

报告CPU相关的统计信息

不加任何参数，表明报告自系统启动以来CPU的平均使用情况。
```
[renyl@localhost kernel]$ mpstat -P ALL
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	03/30/2017 	_x86_64_	(4 CPU)

11:16:08 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
11:16:08 PM  all    0.97    0.04    0.38    0.14    0.04    0.02    0.00    0.00    0.00   98.41
11:16:08 PM    0    1.12    0.02    0.43    0.16    0.03    0.02    0.00    0.00    0.00   98.22
11:16:08 PM    1    0.94    0.01    0.41    0.20    0.09    0.02    0.00    0.00    0.00   98.32
11:16:08 PM    2    0.89    0.07    0.34    0.12    0.02    0.02    0.00    0.00    0.00   98.55
11:16:08 PM    3    0.93    0.05    0.33    0.10    0.02    0.02    0.00    0.00    0.00   98.56
[renyl@localhost kernel]$
```

-I: 报告CPU中断数据
```
[renyl@localhost kernel]$ mpstat -I ALL | tail -5
11:18:06 PM  CPU       HI/s    TIMER/s   NET_TX/s   NET_RX/s    BLOCK/s BLOCK_IOPOLL/s  TASKLET/s    SCHED/s  HRTIMER/s      RCU/s
11:18:06 PM    0       0.00      19.46       0.00       0.02       9.46           0.00       5.96      10.89       0.00       8.10
11:18:06 PM    1       0.00      17.42       0.00       0.39      10.58           0.00       6.69      10.04       0.00       7.40
11:18:06 PM    2       0.00      16.61       0.00       0.04       3.10           0.00       2.98      10.69       0.00       7.49
11:18:06 PM    3       0.00      17.45       0.00       0.05       4.33           0.00       2.30      10.67       0.00       7.91
[renyl@localhost kernel]$
```


### vmstat

报告内存，分页，中断，CPU等信息

不加任何参数，表明报告自系统启动以来的平均统计数据
```
[renyl@localhost ~]$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 5281480  91748 1244864    0    0    80    15   63  182  2  0 98  0  0
[renyl@localhost ~]$ vmstat  -a
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 5285704 841456 1498588    0    0    66    13   60  174  2  0 98  0  0
[renyl@localhost ~]$
```

* r: 可运行进程的数量
* b：不可中断睡眠状态进程的数量
* swpd: 虚拟内存使用的数量
* si: 每秒从磁盘换入到内存的数量
* so: 每秒从内存交换到磁盘的数量
* bi: 每秒从块设备接受的块数
* bo: 每秒发送到块设备的块数
* in: 每秒中断的数量（包括时钟中断）
* cs: 每秒上下文切换的数量
* active: 活跃内存
* inact: 非活跃内存


-m: 显示SLAB相关信息
```
[renyl@localhost ~]$ sudo vmstat -m  1 1 | head -5
Cache                       Num  Total   Size  Pages
fuse_request                 20     20    400     20
fuse_inode                   21     21    768     21
nf_conntrack_expect           0      0    248     33
nf_conntrack                100    100    320     25
[renyl@localhost ~]$
```
* Num: 当前活动对象的数量
* Total: 可用对象的总数
* Size: 每个对象的大小
* Pages: 至少一个活动对象的分页数量


-f: 显示自启动到现在的fork数量
```
[renyl@localhost ~]$ vmstat  -f
         7204 forks
[renyl@localhost ~]$
```

-s: 显示各种事件计数器和内存统计信息
```
[renyl@localhost ~]$ vmstat  -s
      7868508 K total memory
      1264776 K used memory
      1508976 K active memory
       846632 K inactive memory
      5270544 K free memory
        92656 K buffer memory
      1240532 K swap cache
      1952764 K total swap
            0 K used swap
      1952764 K free swap
        26892 non-nice user cpu ticks
         1205 nice user cpu ticks
         5332 system cpu ticks
      1585046 idle cpu ticks
         1580 IO-wait cpu ticks
          691 IRQ cpu ticks
          381 softirq cpu ticks
            0 stolen cpu ticks
       995554 pages paged in
       218284 pages paged out
            0 pages swapped in
            0 pages swapped out
       978131 interrupts
      2832337 CPU context switches
   1490885470 boot time
         7380 forks
[renyl@localhost ~]$
```


### iostat

报告磁盘使用和CPU使用情况

不加任何参数，表明报告自系统启动以来的平均统计数据
```
[renyl@localhost ~]$ iostat
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.99    0.01    0.27    0.12    0.00   98.62

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.02         0.48         0.00       8484          0
sdb               8.00        74.90        50.40    1313002     883444
loop0             0.01         0.13         0.00       2204          0
loop1             0.00         0.06         0.00       1064          4
dm-0              0.00         0.07         0.00       1168          0

[renyl@localhost ~]$
```


-y: 忽略自系统启动以来的平均统计数据
```
[renyl@localhost ~]$ iostat -y -xtd 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

04/01/2017 02:01:29 PM
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb               0.00     8.00    0.00    3.00     0.00    88.00    58.67     0.00    1.33    0.00    1.33   1.33   0.40
loop0             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
loop1             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

04/01/2017 02:01:30 PM
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
loop0             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
loop1             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
dm-0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

[renyl@localhost ~]$ iostat -y -xtd -p sdb 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

04/01/2017 02:02:30 PM
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb2              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb3              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb1              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

04/01/2017 02:02:31 PM
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb2              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb3              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb1              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

[renyl@localhost ~]$
```


### netstat

报告网络相关的信息

显示路由信息
```
[renyl@localhost ~]$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         0.0.0.0         0.0.0.0         U         0 0          0 tun0
default         v101-ir11d-zcr1 0.0.0.0         UG        0 0          0 enp0s25
10.255.28.0     0.0.0.0         255.255.252.0   U         0 0          0 tun0
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.102.204.69  v101-ir11d-zcr1 255.255.255.255 UGH       0 0          0 enp0s25
192.168.50.0    0.0.0.0         255.255.255.0   U         0 0          0 enp0s25
[renyl@localhost ~]$
```

显示网络接口信息
```
[renyl@localhost ~]$ netstat -i
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
docker0   1500        0      0      0 0             0      0      0      0 BMU
enp0s25   1500    43585      0      0 0         42912      0      0      0 BMRU
lo       65536        0      0      0 0             0      0      0      0 LRU
tun0      1266    34355      0      0 0         39707      0      0      0 MOPRU
[renyl@localhost ~]$
```

显示不同协议数据统计信息
```
[renyl@localhost ~]$ netstat -s | head -20
Ip:
    Forwarding: 1
    75943 total packets received
    0 forwarded
    0 incoming packets discarded
    75879 incoming packets delivered
    80859 requests sent out
    15 outgoing packets dropped
    103 dropped because of missing route
Icmp:
    0 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
    1 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 1
IcmpMsg:
        OutType3: 1
Tcp:
[renyl@localhost ~]$
```

显示处于LISTEN状态的TCP连接
```
[renyl@localhost ~]$ sudo netstat -nltp 1
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1820/cupsd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      23206/sshd
tcp6       0      0 ::1:631                 :::*                    LISTEN      1820/cupsd
tcp6       0      0 :::22                   :::*                    LISTEN      23206/sshd
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1820/cupsd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      23206/sshd
tcp6       0      0 ::1:631                 :::*                    LISTEN      1820/cupsd
tcp6       0      0 :::22                   :::*                    LISTEN      23206/sshd
^C
[renyl@localhost ~]$
```


### ss

报告网络相关的信息

显示统计信息
```
[renyl@localhost ~]$ ss -s
Total: 1108 (kernel 0)
TCP:   11 (estab 6, closed 1, orphaned 0, synrecv 0, timewait 1/0), ports 0

Transport Total     IP        IPv6
*	  0         -         -
RAW	  1         0         1
UDP	  14        11        3
TCP	  10        8         2
INET	  25        19        6
FRAG	  0         0         0
[renyl@localhost ~]$
```


显示网络连接信息
```
[renyl@localhost ~]$ ss -t -a | head -10
State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port
LISTEN     0      5      127.0.0.1:ipp                      *:*
LISTEN     0      128        *:ssh                          *:*
ESTAB      0      0      10.255.29.104:42098                10.239.4.160:apex-edge
ESTAB      0      0      10.255.29.104:46712                10.239.4.80:apex-edge
ESTAB      0      0      10.255.29.104:59328                10.239.97.14:snapenetio
ESTAB      0      0      10.255.29.104:46702                10.239.4.80:apex-edge
ESTAB      0      0      192.168.50.27:59850                192.102.204.69:https
SYN-SENT   0      1      10.255.29.104:49988                8.43.85.67:http
ESTAB      0      0      10.255.29.104:46700                10.239.4.80:apex-edge
[renyl@localhost ~]$ ss -t -a -m -p | head -10
State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port
LISTEN     0      5      127.0.0.1:ipp                      *:*
	 skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN     0      128        *:ssh                          *:*
	 skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
ESTAB      0      0      10.255.29.104:42098                10.239.4.160:apex-edge             users:(("chrome",pid=2752,fd=125))
	 skmem:(r0,rb335040,t0,tb46080,f0,w0,o0,bl0)
ESTAB      0      0      10.255.29.104:46712                10.239.4.80:apex-edge             users:(("firefox",pid=3409,fd=74))
	 skmem:(r0,rb335040,t0,tb46080,f4096,w0,o0,bl0)
ESTAB      0      0      10.255.29.104:59328                10.239.97.14:snapenetio            users:(("ssh",pid=11009,fd=3))
[renyl@localhost ~]$
```
* -a: 显示所有监听和非监听的socket
* -t: 显示TCP socket
* -m: 显示socket的内存使用情况
* -p: 显示进程使用的socket


### sar

报告系统CPU，内存，磁盘，网络等信息

不加任何参数，表明报告系统自启动以来每10分钟的CPU使用情况
```
[renyl@localhost lkp-tests]$ sar | head -10
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

09:08:54 AM       LINUX RESTART	(4 CPU)

09:10:00 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
09:20:00 AM     all      4.30      0.09      1.17      0.44      0.00     93.99
09:30:00 AM     all      1.34      0.00      0.38      0.16      0.00     98.12
09:40:00 AM     all      0.15      0.00      0.08      0.03      0.00     99.73
09:50:00 AM     all      0.17      0.00      0.09      0.03      0.00     99.71
10:00:00 AM     all      0.17      0.00      0.08      0.03      0.00     99.72
[renyl@localhost lkp-tests]$
```

显示详细的CPU使用信息
```
[renyl@localhost lkp-tests]$ sar -u ALL -P ALL 1 2 | head -15
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:39:05 PM     CPU      %usr     %nice      %sys   %iowait    %steal      %irq     %soft    %guest    %gnice     %idle
02:39:06 PM     all      0.50      0.00      0.25      0.25      0.00      0.00      0.25      0.00      0.00     98.75
02:39:06 PM       0      1.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00     99.00
02:39:06 PM       1      1.01      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00     98.99
02:39:06 PM       2      0.00      0.00      0.00      1.00      0.00      0.00      0.00      0.00      0.00     99.00
02:39:06 PM       3      1.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00     99.00

02:39:06 PM     CPU      %usr     %nice      %sys   %iowait    %steal      %irq     %soft    %guest    %gnice     %idle
02:39:07 PM     all      0.75      0.00      0.25      0.00      0.00      0.00      0.00      0.00      0.00     99.00
02:39:07 PM       0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00    100.00
02:39:07 PM       1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00    100.00
02:39:07 PM       2      1.01      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00     98.99
02:39:07 PM       3      2.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00     98.00
[renyl@localhost lkp-tests]$
```

显示分页统计信息
```
[renyl@localhost ~]$ sar -B 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:41:11 PM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s    %vmeff
02:41:12 PM      0.00      0.00    395.00      0.00    309.00      0.00      0.00      0.00      0.00
02:41:13 PM      0.00      0.00     59.00      0.00     53.00      0.00      0.00      0.00      0.00
Average:         0.00      0.00    227.00      0.00    181.00      0.00      0.00      0.00      0.00
[renyl@localhost ~]$
```
* pgpgin: 每秒系统从磁盘读入分页的总量（KB）
* pgpgout: 每秒系统移出分页到磁盘的总量（KB）
* faluts: 每秒系统产生分页错误（major+minor)的数量
* majflt: 每秒系统产生主要错误的数量
* pgfree: 每秒系统放置在空闲列表上的分页数量
* pgscank: 每秒kswapd守护进程扫描的分页数量
* pgscand：每秒直接扫描的分页数量
* pgsteal: 每秒系统从缓存回收的分页数量
* vmeff: pgsteal/pgscan计算，分页回收效率的一个度量


显示中断统计信息
```
[renyl@localhost ~]$ sar -I ALL 1 2 | head -10
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:53:32 PM      INTR    intr/s
02:53:33 PM         0      0.00
02:53:33 PM         1      0.00
02:53:33 PM         2      0.00
02:53:33 PM         3      0.00
02:53:33 PM         4      0.00
02:53:33 PM         5      0.00
02:53:33 PM         6      0.00
[renyl@localhost ~]$
```


显示电源管理统计信息
```
[renyl@localhost ~]$ sar -m ALL -P ALL 1  2  | head -20
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:55:31 PM     CPU       MHz
02:55:33 PM     all   1917.34
02:55:33 PM       0   2200.13
02:55:33 PM       1   1821.74
02:55:33 PM       2   1847.52
02:55:33 PM       3   1799.98

02:55:31 PM     FAN       rpm      drpm               DEVICE
02:55:33 PM       1      0.00      0.00    thinkpad-isa-0000

02:55:31 PM    TEMP      degC     %temp               DEVICE
02:55:33 PM       1     39.00     37.14    coretemp-isa-0000
02:55:33 PM       2     39.00     37.14    coretemp-isa-0000
02:55:33 PM       3     39.00     37.14    coretemp-isa-0000
02:55:33 PM       4     39.00      0.00     acpitz-virtual-0
02:55:33 PM       5     45.50      0.00  pch_wildcat_point-v

02:55:31 PM      IN       inV       %in               DEVICE
[renyl@localhost ~]$
```


显示网络接口统计信息
```
[renyl@localhost ~]$ sar -n DEV 1 2 | head -10
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:58:05 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
02:58:06 PM    wlp3s0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
02:58:06 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
02:58:06 PM      tun0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
02:58:06 PM   enp0s25      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
02:58:06 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

02:58:06 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
[renyl@localhost ~]$
```

显示运行队列和平均负载
```
[renyl@localhost ~]$ sar -q 1
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

02:59:44 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
02:59:45 PM         0       689      0.00      0.02      0.00         0
02:59:46 PM         0       689      0.00      0.02      0.00         0
^C

02:59:46 PM         0       689      0.00      0.02      0.00         0
Average:            0       689      0.00      0.02      0.00         0
[renyl@localhost ~]$
```

runq-sz: 运行队列的长度
plist-sz: 在任务列表中的数量
ldavg-x: 过去1，5，15分钟的系统平均负载
blocked: 被blocked的进程数量，等待IO完成


显示内存统计数据
```
[renyl@localhost ~]$ sar -R 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:13:26 PM   frmpg/s   bufpg/s   campg/s
03:13:27 PM     19.00      1.00      0.00
03:13:28 PM      0.00      2.00     10.00
Average:         9.50      1.50      5.00
[renyl@localhost ~]$
```
* frmpg/s: 每秒系统释放内存分页的数量, 负值表示系统分配分页的数量
* bufpg/s: 每秒系统使用额外内存作为缓冲区的数量
* campg/s: 每秒系统使用额外内存作为缓存的数量


显示内存使用情况
```
[renyl@localhost ~]$ sar -r 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:17:41 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
03:17:42 PM   3449616   4418880     56.16    279272   1995852   7561916     77.00   2488516   1549896         0
03:17:43 PM   3449692   4418804     56.16    279272   1995852   7561916     77.00   2488516   1549896        80
Average:      3449654   4418842     56.16    279272   1995852   7561916     77.00   2488516   1549896        40
[renyl@localhost ~]$
```
* kbcommit: 当前工作负载所需要的内存数量
* %commit: 当前工作负载所需要的内存占总内存（RAM + SWAP)的百分比


显示inode状态，文件状态
```
[renyl@localhost ~]$ sar -v 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:26:39 PM dentunusd   file-nr  inode-nr    pty-nr
03:26:40 PM    165837     11808    127310        11
03:26:41 PM    165837     11808    127310        11
Average:       165837     11808    127310        11
[renyl@localhost ~]$
```
* dentunusd: 在目录缓存中未使用的缓存条目的数量
* file-nr: 系统使用的文件处理程序的数量
* inode-nr: 系统使用的inode处理程序的数量
* pyt-nr: 系统使用伪终端的数量


显示SWAP的统计数据
```
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:30:46 PM  pswpin/s pswpout/s
03:30:47 PM      0.00      0.00
03:30:48 PM      0.00      0.00
Average:         0.00      0.00
[renyl@localhost ~]$
```

显示任务创建和系统切换统计数据
```
[renyl@localhost ~]$ sar -w 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:32:01 PM    proc/s   cswch/s
03:32:02 PM      1.00    525.00
03:32:03 PM      0.00    366.00
Average:         0.50    445.50
[renyl@localhost ~]$
```
* proc/s: 没秒创建任务的总数
* cswch/s: 每秒context切换的总数


显示hugepages使用情况
```
[renyl@localhost ~]$ sar -H 1 2
Linux 4.8.14-100.fc23.x86_64 (localhost.localdomain) 	04/01/2017 	_x86_64_	(4 CPU)

03:33:40 PM kbhugfree kbhugused  %hugused
03:33:41 PM         0         0      0.00
03:33:42 PM         0         0      0.00
Average:            0         0      0.00
[renyl@localhost ~]$
```


### numastat

显示numa架构中每个node的内存相关信息

```
[renyl@localhost ~]$ numastat
                           node0           node1
numa_hit            108056409671     87351704351
numa_miss             1006092384      1267735281
numa_foreign          1267735281      1006092076
interleave_hit          21081924        20389267
local_node          108044118004     87343901565
other_node            1018384051      1275538067
[renyl@localhost ~]$ numastat -mn | head -10

Per-node system memory usage (in MBs):
                          Node 0          Node 1           Total
                 --------------- --------------- ---------------
MemTotal                96639.66        96743.38       193383.05
MemFree                   464.75          208.77          673.52
MemUsed                 96174.92        96534.61       192709.53
Active                  79583.53        77643.71       157227.24
Inactive                 8026.04         8540.91        16566.95
Active(anon)            72643.86        69232.95       141876.81
[renyl@localhost ~]$ numastat -mn  -p 111341 | head -20

Per-node process memory usage (in MBs) for PID 111341 (ruby)
                           Node 0          Node 1           Total
                  --------------- --------------- ---------------
Huge                         0.00            0.00            0.00
Heap                         1.44            8.06            9.50
Stack                        0.00            0.05            0.05
Private                      4.30            0.72            5.02
----------------  --------------- --------------- ---------------
Total                        5.74            8.83           14.57

Per-node system memory usage (in MBs):
                          Node 0          Node 1           Total
                 --------------- --------------- ---------------
MemTotal                96639.66        96743.38       193383.05
MemFree                   444.31          273.79          718.10
MemUsed                 96195.35        96469.59       192664.95
Active                  79774.78        77779.09       157553.88
Inactive                 7875.56         8306.68        16182.25
Active(anon)            72864.18        69101.00       141965.18
[renyl@localhost ~]$
```


### pmap

显示进程的内存映射

```
[renyl@localhost ~]$ pmap -p 2305 | head -10
2305:   /usr/libexec/gnome-terminal-server
00005577359a0000    316K r-x-- /usr/libexec/gnome-terminal-server
0000557735bee000     20K r---- /usr/libexec/gnome-terminal-server
0000557735bf3000      4K rw--- /usr/libexec/gnome-terminal-server
0000557735bf4000      4K rw---   [ anon ]
000055773796e000  28276K rw---   [ anon ]
00007f2178000000    136K rw---   [ anon ]
00007f2178022000  65400K -----   [ anon ]
00007f2180000000    136K rw---   [ anon ]
00007f2180022000  65400K -----   [ anon ]
[renyl@localhost ~]$ pmap -d 2305 | head -10
2305:   /usr/libexec/gnome-terminal-server
Address           Kbytes Mode  Offset           Device    Mapping
00005577359a0000     316 r-x-- 0000000000000000 008:00013 gnome-terminal-server
0000557735bee000      20 r---- 000000000004e000 008:00013 gnome-terminal-server
0000557735bf3000       4 rw--- 0000000000053000 008:00013 gnome-terminal-server
0000557735bf4000       4 rw--- 0000000000000000 000:00000   [ anon ]
000055773796e000   28276 rw--- 0000000000000000 000:00000   [ anon ]
00007f2178000000     136 rw--- 0000000000000000 000:00000   [ anon ]
00007f2178022000   65400 ----- 0000000000000000 000:00000   [ anon ]
00007f2180000000     136 rw--- 0000000000000000 000:00000   [ anon ]
[renyl@localhost ~]$ pmap -x 2305 | head -10
2305:   /usr/libexec/gnome-terminal-server
Address           Kbytes     RSS   Dirty Mode  Mapping
00005577359a0000     316     316       0 r-x-- gnome-terminal-server
00005577359a0000       0       0       0 r-x-- gnome-terminal-server
0000557735bee000      20      20      20 r---- gnome-terminal-server
0000557735bee000       0       0       0 r---- gnome-terminal-server
0000557735bf3000       4       4       4 rw--- gnome-terminal-server
0000557735bf3000       0       0       0 rw--- gnome-terminal-server
0000557735bf4000       4       4       4 rw---   [ anon ]
0000557735bf4000       0       0       0 rw---   [ anon ]
[renyl@localhost ~]$
```


### strace

跟追系统调用

基本用法
```
[renyl@localhost ~]$ strace -t ls 2>&1 | head -10
15:49:58 execve("/usr/bin/ls", ["ls"], [/* 52 vars */]) = 0
15:49:58 brk(NULL)                      = 0x56048b73c000
15:49:58 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f2dd88c1000
15:49:58 access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
15:49:58 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
15:49:58 fstat(3, {st_mode=S_IFREG|0644, st_size=115013, ...}) = 0
15:49:58 mmap(NULL, 115013, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f2dd88a4000
15:49:58 close(3)                       = 0
15:49:58 open("/lib64/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
15:49:58 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\260b\0\0\0\0\0\0"..., 832) = 832
[renyl@localhost ~]$
```

跟踪指定的系统调用
```
calhost ~]$ strace -t -e trace=read ls
15:53:12 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\260b\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0p\26\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\240\6\2\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\320\26\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0`\16\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\320\23\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\300`\0\0\0\0\0\0"..., 832) = 832
15:53:12 read(3, "nodev\tsysfs\nnodev\trootfs\nnodev\tr"..., 1024) = 388
15:53:12 read(3, "", 1024)              = 0
Allen  bin  Desktop  docker  Downloads	lkp  repo  skydata  skydata-ci	SkyDiscovery  testdir
15:53:12 +++ exited with 0 +++
[renyl@localhost ~]$
```
* -e trace=network: 跟踪所有网络相关的系统调用
* -e trace=ipc: 跟踪所有IPC相关的系统调用
* -e trace=signal: 跟踪所有信号相关的系统调用


显示每个系统调用的时间、调用次数
```
[renyl@localhost ~]$ strace -c ls
Allen  bin  Desktop  docker  Downloads	lkp  repo  skydata  skydata-ci	SkyDiscovery  testdir
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         9           read
  0.00    0.000000           0         1           write
  0.00    0.000000           0        11           open
  0.00    0.000000           0        13           close
  0.00    0.000000           0         1           stat
  0.00    0.000000           0        12           fstat
  0.00    0.000000           0        27           mmap
  0.00    0.000000           0        16           mprotect
  0.00    0.000000           0         3           munmap
  0.00    0.000000           0         3           brk
  0.00    0.000000           0         2           rt_sigaction
  0.00    0.000000           0         1           rt_sigprocmask
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         2         1 access
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         2           getdents
  0.00    0.000000           0         1           getrlimit
  0.00    0.000000           0         2         2 statfs
  0.00    0.000000           0         1           arch_prctl
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0         1           set_robust_list
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                   112         3 total
[renyl@localhost ~]$
```


### ltrace

跟踪库调用

基本用法
```
[renyl@localhost ~]$ ltrace -t ls 2>&1 | head -10
16:38:30 malloc(552)                             = 0x1da8010
16:38:30 malloc(120)                             = 0x1da8240
16:38:30 malloc(1024)                            = 0x1da82c0
16:38:30 free(0x1da82c0)                         = <void>
16:38:30 free(0x1da8010)                         = <void>
16:38:30 __libc_start_main(0x4028a0, 1, 0x7ffe0df89ef8, 0x4124f0 <unfinished ...>
16:38:30 strrchr("ls", '/')                      = nil
16:38:30 setlocale(LC_ALL, "" <unfinished ...>
16:38:30 malloc(5)                               = 0x1da8010
16:38:30 free(0x1da8010)                         = <void>
[renyl@localhost ~]$
```

跟踪指定的库调用
```
[renyl@localhost ~]$ ltrace -t -e malloc ls 2>&1 | head -10
16:40:22 ls->malloc(552)                         = 0x1656010
16:40:22 ls->malloc(120)                         = 0x1656240
16:40:22 ls->malloc(1024)                        = 0x16562c0
16:40:22 ls->malloc(5)                           = 0x1656010
16:40:22 ls->malloc(120)                         = 0x1656030
16:40:22 ls->malloc(12)                          = 0x1656010
16:40:22 ls->malloc(776)                         = 0x16560b0
16:40:22 ls->malloc(112)                         = 0x16563c0
16:40:22 ls->malloc(952)                         = 0x1656440
16:40:22 ls->malloc(216)                         = 0x1656800
[renyl@localhost ~]$
```

显示库调用的时间、调用次数
```
[renyl@localhost ~]$ ltrace -c ls
Allen  bin  Desktop  docker  Downloads  lkp  repo  skydata  skydata-ci  SkyDiscovery  testdir
% time     seconds  usecs/call     calls      function
------ ----------- ----------- --------- --------------------
 22.33    0.008861         104        85 __errno_location
 15.48    0.006143          73        84 __ctype_get_mb_cur_max
 10.77    0.004273         170        25 memcpy
 10.10    0.004009          71        56 malloc
  8.09    0.003211          80        40 strcoll
  7.31    0.002900          78        37 readdir
  7.09    0.002815        2815         1 setlocale
  4.46    0.001769          70        25 __overflow
  2.90    0.001149          76        15 strlen
  2.69    0.001066          76        14 fwrite_unlocked
  2.00    0.000794          72        11 free
  1.38    0.000548          68         8 getenv
  0.70    0.000276          69         4 __freading
  0.59    0.000234         117         2 fclose
  0.52    0.000208         208         1 closedir
  0.46    0.000184         184         1 opendir
  0.38    0.000150         150         1 bindtextdomain
  0.38    0.000150          75         2 fileno
  0.36    0.000141         141         1 textdomain
  0.34    0.000136          68         2 fflush
  0.33    0.000132          66         2 __fpending
  0.22    0.000089          89         1 realloc
  0.20    0.000079          79         1 strrchr
  0.20    0.000079          79         1 _setjmp
  0.20    0.000079          79         1 isatty
  0.19    0.000076          76         1 getopt_long
  0.18    0.000073          73         1 ioctl
  0.16    0.000063          63         1 __cxa_atexit
------ ----------- ----------- --------- --------------------
100.00    0.039687                   424 total

[renyl@localhost ~]$
```

### turbostat

显示CPU的C-state信息

```
[renyl@localhost ~]$ turbostat -i 1
[1491037400.730710] cpu0, energy_pkg_last=0x1ad11594
[1491037401.731231] cpu0, pkg_last=0x1ad11594, pkg=0x1ad7b41d, delta=0x00069e89
energy_pkg=0x69e89, units=0x0.000015, interval=1.000502
energy_pkg=0x69e89, units=0x0.000015, interval=1.000502
cor CPU    %c0  GHz  TSC SMI    %c1    %c3    %c6    %c7 CTMP PTMP   %pc2   %pc3   %pc6   %pc7  Pkg_W  Cor_W GFX_W
          3.22 1.70 3.39   0   6.34   0.15  90.28   0.00   54   54  26.12   0.15  56.82   0.00   6.62   2.54  0.30
  0   0   3.37 1.64 3.39   0   6.56   0.46  89.61   0.00   54   54  26.12   0.15  56.82   0.00   6.62   2.54  0.30
  0   4   3.69 1.79 3.39   0   6.24
  1   1   3.17 1.68 3.39   0   6.19   0.10  90.55   0.00   54
  1   5   3.13 1.67 3.39   0   6.23
  2   2   3.21 1.78 3.39   0   6.23   0.03  90.53   0.00   54
  2   6   2.95 1.65 3.39   0   6.49
  3   3   3.19 1.71 3.39   0   6.34   0.04  90.43   0.00   54
  3   7   3.07 1.65 3.39   0   6.47
[1491037401.731664] cpu0, energy_pkg_last=0x1ad7b82e
[1491037402.732051] cpu0, pkg_last=0x1ad7b82e, pkg=0x1ae2f05c, delta=0x000b382e
energy_pkg=0xb382e, units=0x0.000015, interval=1.000408
energy_pkg=0xb382e, units=0x0.000015, interval=1.000408
cor CPU    %c0  GHz  TSC SMI    %c1    %c3    %c6    %c7 CTMP PTMP   %pc2   %pc3   %pc6   %pc7  Pkg_W  Cor_W GFX_W
         10.94 3.01 3.39   0   5.92   0.08  83.06   0.00   64   64  23.98   0.03  52.57   0.00  11.21   7.16  0.30
  0   0  11.29 3.01 3.39   0   5.76   0.01  82.94   0.00   64   64  23.98   0.03  52.57   0.00  11.21   7.16  0.30
  0   4  11.01 2.99 3.39   0   6.04
  1   1  11.01 3.02 3.39   0   5.86   0.12  83.02   0.00   62
  1   5  10.93 3.01 3.39   0   5.94
  2   2  10.74 3.02 3.39   0   6.09   0.06  83.11   0.00   63
  2   6  10.91 3.04 3.39   0   5.92
  3   3  10.86 3.01 3.39   0   5.84   0.14  83.16   0.00   62
  3   7  10.77 3.01 3.39   0   5.94
[1491037402.732453] cpu0, energy_pkg_last=0x1ae2f05c
^C
[renyl@localhost ~]$
```

### perf

Linux下性能分析tool

列出所有的事件
```
[renyl@localhost ~]$ perf  list | head -10
  branch-instructions OR branches                    [Hardware event]
  branch-misses                                      [Hardware event]
  bus-cycles                                         [Hardware event]
  cache-misses                                       [Hardware event]
  cache-references                                   [Hardware event]
  cpu-cycles OR cycles                               [Hardware event]
  instructions                                       [Hardware event]
  ref-cycles                                         [Hardware event]
  alignment-faults                                   [Software event]
  bpf-output                                         [Software event]
[renyl@localhost ~]$ 
```

统计cache-misses事件
```
[renyl@localhost ~]$ perf stat -e cache-misses ls
Allen  bin  Desktop  docker  Downloads	lkp  perf.log  repo  skydata  skydata-ci  SkyDiscovery	testdir

 Performance counter stats for 'ls':

             7,812      cache-misses:u                                              

       0.001261013 seconds time elapsed

[renyl@localhost ~]$ 
```

采集并查看详细的性能统计
```
[renyl@localhost ~]$ perf record -g -o perf.log ls
Allen  bin  Desktop  docker  Downloadslkp  perf.log  repo  skydata  skydata-ci  SkyDiscoverytestdir
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.017 MB perf.log (15 samples) ]
[renyl@localhost ~]$ perf report -g -i perf.log
Samples: 15  of event 'cycles:u', Event count (approx.): 2033512
  Children      Self  Command  Shared Object      Symbol
+   44.86%     0.00%  ls       ld-2.22.so         [.] _dl_sysdep_start
+   39.29%     0.00%  ls       ld-2.22.so         [.] dl_main
+   31.80%     0.00%  ls       [unknown]          [.] 0000000000000000
+   22.12%    22.12%  ls       ld-2.22.so         [.] do_lookup_x
+   20.15%    20.15%  ls       ld-2.22.so         [.] _dl_lookup_symbol_x
+   15.97%    15.97%  ls       libc-2.22.so       [.] malloc_consolidate
+   15.83%     0.00%  ls       ld-2.22.so         [.] _dl_fini
+   15.83%     0.00%  ls       libc-2.22.so       [.] __run_exit_handlers
+   15.83%     0.00%  ls       libc-2.22.so       [.] 0xffff80e8c6ac9080
+   15.83%    15.83%  ls       ld-2.22.so         [.] memset
+   13.75%     0.00%  ls       [unknown]          [.] 0xffae4acebe35cada
+    9.85%     9.85%  ls       ld-2.22.so         [.] openaux
+    9.29%     9.29%  ls       libc-2.22.so       [.] strchr
+    8.66%     0.00%  ls       ld-2.22.so         [.] _dl_relocate_object
+    5.57%     0.00%  ls       ld-2.22.so         [.] _dl_init_paths
+    5.57%     5.57%  ls       [kernel.kallsyms]  [k] page_fault
+    1.21%     1.21%  ls       ld-2.22.so         [.] _start
renyl@localhost ~]$
```


### virt-top / xentop

显示kvm/xen的CPU和memory信息

```
virt-top –b –d $interval –n $count
xentop  –b –d $interval –n $count
```
* virt-top: 采集集的是机器上所有cpu的平均利用率（A%），想要计算出guest的cpu平均利用率，需要进行如下计算：B%=A% *（总共cpu/分配给Guest的cpu）
* xentop采集的是分配给Guest的cpu的总利用率


## proc文件系统

### cpuinfo

查看CPU信息
```
[renyl@localhost ~]$ cat /proc/cpuinfo | head -20
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 61
model name	: Intel(R) Core(TM) i5-5200U CPU @ 2.20GHz
stepping	: 4
microcode	: 0x24
cpu MHz		: 2446.936
cache size	: 3072 KB
physical id	: 0
siblings	: 4
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 20
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer xsave avx f16c rdrand lahf_lm abm 3dnowprefetch epb intel_pt tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx smap xsaveopt dtherm ida arat pln pts
[renyl@localhost ~]$
```

### meminfo

查看内存信息
```
[renyl@localhost ~]$ cat /proc/meminfo | head -20
MemTotal:        7868496 kB
MemFree:         3341496 kB
MemAvailable:    4645320 kB
Buffers:          295584 kB
Cached:          2008192 kB
SwapCached:            0 kB
Active:          2592072 kB
Inactive:        1546732 kB
Active(anon):    1837056 kB
Inactive(anon):   876996 kB
Active(file):     755016 kB
Inactive(file):   669736 kB
Unevictable:         136 kB
Mlocked:             136 kB
SwapTotal:       1952764 kB
SwapFree:        1952764 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:       1835216 kB
Mapped:           735516 kB
[renyl@localhost ~]$
```


### cmdline

查看Linux启动命令行
```
[renyl@localhost ~]$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-4.8.14-100.fc23.x86_64 root=UUID=0c6972b9-0239-4460-8397-5a944ad58f88 ro rhgb quiet LANG=en_US.UTF-8
[renyl@localhost ~]$
```


### buddyinfo

查看内存中，每个区域中有多少内存块可用
```
[renyl@localhost ~]$ cat /proc/buddyinfo
Node 0, zone      DMA      1      0      0      0      1      1      1      0      1      1      3
Node 0, zone    DMA32      1      2      4      2      4      2      4      4      4      2    613
Node 0, zone   Normal    261    686    337    168     90     31     18      9      3      4    675
[renyl@localhost ~]$
```
每一列的数量意味着可用的分页数量。Linux采用伙伴系统（buddy system）来维护空闲分页，因此，
Normal区的第0列可分配空间为：261 * 2^0 * 4KB, 第1列可分配空间为： 686 * 2^1 * 4KB，以此类推。


### partitions

显示分区信息
```
[renyl@localhost ~]$ cat /proc/partitions 
major minor  #blocks  name

   8        0  488386584 sda
   8        1    1536000 sda1
   8        2  466368512 sda2
   8        3   20480000 sda3
   8       16  125034840 sdb
   8       17     975872 sdb1
   8       18    1952768 sdb2
   8       19  122104832 sdb3
   7        0  104857600 loop0
   7        1    2097152 loop1
 253        0  104857600 dm-0
[renyl@localhost ~]$
```

### interrupts

显示中断统计信息
```
[renyl@localhost ~]$ cat /proc/interrupts  | head -10
           CPU0       CPU1       CPU2       CPU3
  0:         48          0          0          0  IR-IO-APIC   2-edge      timer
  1:          3          1          4          1  IR-IO-APIC   1-edge      i8042
  8:          0          1          0          0  IR-IO-APIC   8-edge      rtc0
  9:        647       5021        160        215  IR-IO-APIC   9-fasteoi   acpi
 12:        104        205         26        176  IR-IO-APIC  12-edge      i8042
 18:          1          0          0          0  IR-IO-APIC  18-fasteoi   i801_smbus
 23:          3         19          5          6  IR-IO-APIC  23-fasteoi   ehci_hcd:usb1
 40:          0          0          0          0  DMAR-MSI   0-edge      dmar0
 41:          0          0          0          0  DMAR-MSI   1-edge      dmar1
[renyl@localhost ~]$
```

### smp_affinity_list

显示中断号对应的CPU
```
[renyl@localhost ~]$ cat /proc/irq/11/smp_affinity_list
0-7
[renyl@localhost ~]$
```
表示11号中断可以在0-7号CPU上产生硬件中断。


### version

显示系统版本
```
[renyl@localhost ~]$ cat /proc/version
Linux version 4.8.14-100.fc23.x86_64 (mockbuild@bkernel01.phx2.fedoraproject.org) (gcc version 5.3.1 20160406 (Red Hat 5.3.1-6) (GCC) ) #1 SMP Mon Dec 12 20:52:15 UTC 2016
[renyl@localhost ~]$
```

### pid

显示进程的各种信息
```
[renyl@localhost ~]$ ls /proc/2129/
attr       cgroup      comm             cwd      fd       io       loginuid   mem        mountstats  numa_maps  oom_score_adj  projid_map  schedstat  smaps  statm    task           uid_map
autogroup  clear_refs  coredump_filter  environ  fdinfo   latency  map_files  mountinfo  net         oom_adj    pagemap        root        sessionid  stack  status   timers         wchan
auxv       cmdline     cpuset           exe      gid_map  limits   maps       mounts     ns          oom_score  personality    sched       setgroups  stat   syscall  timerslack_ns
[renyl@localhost ~]$
```
* cmdline: 启动命令行
* comm: 程序名字
* oom_score_adj: 可以调正OOM的优先级
* maps: 内存映射
* stack: 栈调用


### vm

各种跟内存相关的设置
```
[renyl@localhost proc]$ ls /proc/sys/vm/
admin_reserve_kbytes         dirty_bytes                extfrag_threshold           max_map_count              mmap_min_addr            nr_pdflush_threads        overcommit_ratio          swappiness
block_dump                   dirty_expire_centisecs     hugepages_treat_as_movable  memory_failure_early_kill  mmap_rnd_bits            numa_zonelist_order       page-cluster              user_reserve_kbytes
compact_memory               dirty_ratio                hugetlb_shm_group           memory_failure_recovery    mmap_rnd_compat_bits     oom_dump_tasks            panic_on_oom              vfs_cache_pressure
compact_unevictable_allowed  dirtytime_expire_seconds   laptop_mode                 min_free_kbytes            nr_hugepages             oom_kill_allocating_task  percpu_pagelist_fraction  watermark_scale_factor
dirty_background_bytes       dirty_writeback_centisecs  legacy_va_layout            min_slab_ratio             nr_hugepages_mempolicy   overcommit_kbytes         stat_interval             zone_reclaim_mode
dirty_background_ratio       drop_caches                lowmem_reserve_ratio        min_unmapped_ratio         nr_overcommit_hugepages  overcommit_memory         stat_refresh
[renyl@localhost proc]$
```

### net

各种跟网络相关的设置
```
[renyl@localhost proc]$ ls /proc/sys/net/
bridge  core  ipv4  ipv6  netfilter  nf_conntrack_max  unix
[renyl@localhost proc]$ ls /proc/sys/net/core/
bpf_jit_enable          busy_read               flow_limit_cpu_bitmap   message_burst           netdev_max_backlog      optmem_max              rps_sock_flow_entries   warnings                xfrm_acq_expires        xfrm_larval_drop
bpf_jit_harden          default_qdisc           flow_limit_table_len    message_cost            netdev_rss_key          rmem_default            somaxconn               wmem_default            xfrm_aevent_etime
busy_poll               dev_weight              max_skb_frags           netdev_budget           netdev_tstamp_prequeue  rmem_max                tstamp_allow_data       wmem_max                xfrm_aevent_rseqth
[renyl@localhost proc]$ ls /proc/sys/net/core/
bpf_jit_enable  busy_poll  default_qdisc  flow_limit_cpu_bitmap  max_skb_frags  message_cost   netdev_max_backlog  netdev_tstamp_prequeue  rmem_default  rps_sock_flow_entries  tstamp_allow_data  wmem_default  xfrm_acq_expires   xfrm_aevent_rseqth
bpf_jit_harden  busy_read  dev_weight     flow_limit_table_len   message_burst  netdev_budget  netdev_rss_key      optmem_max              rmem_max      somaxconn              warnings           wmem_max      xfrm_aevent_etime  xfrm_larval_drop
[renyl@localhost proc]$
```


### kernel

各种核心参数的设置
```
[renyl@localhost proc]$ ls /proc/sys/kernel/
acct                ftrace_dump_on_oops           msgmax                             overflowgid                        perf_event_paranoid      sched_cfs_bandwidth_slice_us  sched_wakeup_granularity_ns   sysrq
acpi_video_flags    ftrace_enabled                msgmnb                             overflowuid                        pid_max                  sched_child_runs_first        sem                           tainted
auto_msgmni         hardlockup_all_cpu_backtrace  msgmni                             panic                              poweroff_cmd             sched_domain                  sem_next_id                   threads-max
bootloader_type     hardlockup_panic              msg_next_id                        panic_on_io_nmi                    print-fatal-signals      sched_latency_ns              sg-big-buff                   timer_migration
bootloader_version  hostname                      ngroups_max                        panic_on_oops                      printk                   sched_migration_cost_ns       shmall                        traceoff_on_warning
cad_pid             io_delay_type                 nmi_watchdog                       panic_on_rcu_stall                 printk_delay             sched_min_granularity_ns      shmmax                        tracepoint_printk
cap_last_cap        kexec_load_disabled           ns_last_pid                        panic_on_stackoverflow             printk_devkmsg           sched_nr_migrate              shmmni                        unknown_nmi_panic
compat-log          keys                          numa_balancing                     panic_on_unrecovered_nmi           printk_ratelimit         sched_rr_timeslice_ms         shm_next_id                   unprivileged_bpf_disabled
core_pattern        kptr_restrict                 numa_balancing_scan_delay_ms       panic_on_warn                      printk_ratelimit_burst   sched_rt_period_us            shm_rmid_forced               usermodehelper
core_pipe_limit     kstack_depth_to_print         numa_balancing_scan_period_max_ms  perf_cpu_time_max_percent          pty                      sched_rt_runtime_us           softlockup_all_cpu_backtrace  version
core_uses_pid       latencytop                    numa_balancing_scan_period_min_ms  perf_event_max_contexts_per_stack  random                   sched_schedstats              softlockup_panic              watchdog
ctrl-alt-del        max_lock_depth                numa_balancing_scan_size_mb        perf_event_max_sample_rate         randomize_va_space       sched_shares_window_ns        soft_watchdog                 watchdog_cpumask
dmesg_restrict      modprobe                      osrelease                          perf_event_max_stack               real-root-dev            sched_time_avg_ms             stack_tracer_enabled          watchdog_thresh
domainname          modules_disabled              ostype                             perf_event_mlock_kb                sched_autogroup_enabled  sched_tunable_scaling         sysctl_writes_strict          yama
[renyl@localhost proc]$
```


## 磁盘认识

1) 一个IO操作基本可以分为两个部分：`数据定位+数据传输`。平均定位时间主要有两部分组成：平均寻道时间和平均转动延迟（由磁盘转速决定）。
数据传输速率取决有磁盘的转速和存储密度。

2)一个IO操作的延迟由三个部分组成：寻道延迟+转动延迟+数据传输延迟。所以，磁盘硬件厂商要提高磁盘速度就要拼命提高转速和密度。
而在软件上，我们可以实现减少平均数据定位的时间来提高磁盘的性能，这就是linux系统关于disk做的一个readahead特性。

3)磁盘有多种类型，它们的不同之处在于：
* IDE(Integrated Drive Electronics):是通过一个叫PATA（Paralle Advanced Technology Attachment）的接口实现的，需要注意的是“并行",
  由于采用并行总线接口，传输数据和信号的总线是复用的，因此传输速率会受到一定的限制。如果要提高传输的速率，那么传输的数据和信号往往会产生干扰，
  从而导致错误，因此性能会受到影响。于是，SATA出现了。

* SATA(Serial ATA)：是通过串行ATA接口实现的，于是又叫“串口硬盘”。这种接口把数据总线和信号总线分开了，数据线和信号线独立使用，因此磁盘的性能提高了。
  但是，ATA接口方式的磁盘转速提升有限，而磁盘的转速对磁盘的性能具有重要的影响。于是， SCSI出现了。

* SCSI(Small Computer System Interface):SCSI接口可以解决的是磁盘的转速问题，从而提高磁盘性能。但是，刚开始，SCSI接口还是并行设计的，
  即传输数据和信号的总线是复用的，这同样会限制磁盘的性能。于是，SAS出现了。

* SAS(Serial Attached SCSI):串行SCSI，这种接口方式又提高了磁盘的性能。由于磁盘内部的机械结构限制，没法再大幅度提供磁盘的性能了。于是，SSD出现了。

* SSD(Solid State Disk):SSD与普通机械磁盘不同，其是由控制单元和固态存储单元组成，不需要在搞什么磁头了等等，性能真正的大大的提高。

4)磁盘的组织方式多样化，很让人晕，简单介绍下：

* RAID：这个就不多介绍了，大家都知道。

* NAS(Network Attached Storage):NAS就像一个存储设备，但是该存储设备有自己的OS，存储处理单元等等，就类似咱门的文件服务器，只要客户端符合其协议，
  大家都可以通过网络到NAS上读写文件。由于，大家都通过网路去访问，因此网络就会出现瓶颈，从而影响IO的性能。于是，SAN出现了。

* SAN(Storage Area Network):SAN一般是通过FC来连接存储设备和主机（如：富士通的ETERNUS和S8），这样给主机提供高的磁盘性能。需要注意的是，SAN相当于一个存储网络，
  SAN中的存储设备没有自己的OS。但是，由于使用了FC，这就得需要特定的存储设备来支持啊，而这个通常来说都非常贵啊。于是，iSCSI就出现了。

* iSCSI:由于现在10GB网卡和交换机也很普遍了，比FC也慢不了多少，而iSCSI的存储设备相对SAN来说却便宜多了，所以用的比较多。iSCSI不再使用FC，就是使用普通的网线，
  通过TCP/IP来传递磁盘数据，数据在传输前需要在前面加个iSCSI头（就类似tcp/ip的头），接受到数据后再按一定格式把这个头解开，虽然有一定的额外开销，
  但是这是避免不了的啊。总体来说，这个目前来说，还是性价比最高的。

4)SCSI和iSCSI有什么关系吗？（它们俩一毛钱都没有）

* SCSI是磁盘的一个接口标准，注意，这是一个接口标准，任何采用该标准的磁盘都叫SCSI磁盘。

* iSCSI是一个供硬件设备使用的可以在IP协议的上层运行的SCSI指令集，这种指令集合可以实现在IP网络上运行SCSI协议。
  因此，我们完全可以使用SATA磁盘（注意，不是SCSI磁盘噢）作为硬件设备，然后通过iSCSI来提供网络存储功能。

5) 如何通过iostat的信息，来对磁盘的性能进行分析。以及iostat中的各个参数之间有什么样的关系。

以下面的输出作为例子：
```
Device:   rkB/s      wkB/s     avgrq-sz    avgqu-sz     await     r_await     w_await    svctm     %util
sda        0.00    5756.00         8.00        0.11      0.08        0.00        0.08     0.08     11.00
```

分析：
* 首先，我们可以判断的是，这是一个写操作的数据，因为 w/s 有数据， r/s 没有数据。
  同时，这个参数就相当于iops，因此能得出这个公式：wkB/s= w/s * avgrq-sz *512.

* 我们可以知道这个写操作的bs大小为 4KB，因为 avgrq-sz为8，逻辑块大小为512B，因此bs=8*512B=4096B=4KB

* 我们看到await和svctm基本一致，说明磁盘操作不需要任何等待，所以基本可以判断IO队列长度为1，
  同时avgqu-sz大小小于1也能证明

* util%的值才11%，说明磁盘利用率不高，利用率不高的原因是由于avgqu-sz过低，
  导致磁盘需要等待队列中的数据。那么如何增加平均队列长度呢？有三种方法
	* 使用异步IO，同时发起多个IO请求，相当于队列中有多个IO请求 （这就如fio中iodepth > 1的作用）

	* 多线程发起同步IO请求，相当于队列中有多个IO请求（这就如我们平常跑的多重度dd的效果）

	* 增大应用IO大小，到达底层之后，会变成多个IO请求，相当于队列中有多个IO请求
	 （这个就等同调大bs，bs过大的话，到底层io调度的时候会被切分，这个值的大小由内核参数/sys/block/sda/queue/max_sectors_kb控制）

