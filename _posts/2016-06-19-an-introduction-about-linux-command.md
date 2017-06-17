---
title: Linux 常用命令简介
category: linux
tags:
- linux 
- command
---

## 介绍

本文将对Linux常用的各种命令进行介绍。


### alias

说明: 为了支持在shell脚本中使用alias命令对其他命令进行重命名
```
shopt –s expand_aliases   #可以这样进行enable
```


### ascii

说明: 得到一个完整的ASCII表，有对应的16进制和10进制的值
```
# man ascii
```


### ausyscall

说明: 查看系统调用名和调用号的映射
```
# ausyscall  --dump
```

<!--more-->

### apt-get

| 序号|	命令| 说明|
| --- | --- | --- |
| 1 | apt-get install gcl |	安装包/更新包 |
| 2 | apt-get install –s gcl | 模拟安装包（simulate）|
| 3 | apt-get install –d gcl | 下载包，但是不安装它。|
| 4 | apt-get clean | 清除下载的包文件|
| 5	| apt-get remove gcl | 删除包|
| 6 | apt-get autoremove gcl |1)同时删除依赖的包 2)如果没有指定包名,删除系统中作为依赖包安装的所有不使用的包|
| 7 | apt-get update  | 更新本地数据库|
| 8 | apt-get upgrade |	命令更新系统上的所有包|

相关配置文件：
1）/etc/apt/sources.list
这个列表告诉 apt-get 到哪里寻找包，包括 CD-ROM、本地文件系统以及使用 http 或 ftp 在网上寻找。
可以在 /etc/apt/sources.list.d 目录中添加更多来源。

2）/etc/apt/apt.conf 
如果您经常使用 apt-get 命令，觉得不喜欢默认选项，可以设置新的默认选项。
可以使用 apt-config 程序访问 apt.conf 文件。关于 apt.conf 和 apt-config 的更多信息参见手册页。

3）/var/cache/apt/archives/
下载的文件常常放在 /var/cache/apt/archives/ 中


### awk

while循环
```
# cat file |　awk '{ i=1;while(i<=NF) {print NF,$i;i++}}' file
```

for循环
```
# cat file |　awk '{ for(i=1;i<NF;i++) {print NF,$i}}' file
```

if语句
```
# cat file |  awk '{ if (NR<3)  print $1;  else print $2 }'
```

求和
```
# cat file | sed -n '1,10p' |  awk '{ print $6 }' | awk 'BEGIN {sum=0}; {sum=sum+$1}; END {print sum}'
```


### bash

说明：可以查看man bash来获取默认键的绑定设置

| 序号 | 快捷键 | 说明 |
| --- | --- | --- |
| 1 | ctrl + r | 通过关键字搜索history文件里的命令 |
| 2 | ctrl + u | 清除整行 |
| 3 | ctrl + a | 回到命令开头 |
| 4 | ctrl + w | 来清除最后一个单词 |
| 5 | alt + . | 遍历前一个命令使用的最后一个参数 |
| 6 | !command | 执行最近一次执行command的命令 |
| 7 | ctrl+shift+t | 开启一个新的tab terminal，使用alt+num进行切换 |
| 8 | ctrl + p | 搜索前一条命令 |
| 9 | ctrl + n | 搜索后一条命令 |
| 10 | alt + b | 回到前一个字符串 |
| 11 | alt + f | 回到后一个字符串 |


### bc

说明：shell里进行小数计算
```
# total=`echo "scale=2;($throughput+$total)" | bc`
```
注：scale表示小数精度。


### chrt

说明：调整进程优先级
```
# chrt -p $pid  //显示进程的优先级属性
# chrt -m      //显示系统支持的最大/最小优先级值
# chrt -f -p 30 $pid
# chrt -f 30 $cmd
```


### cpio

打包压缩：
```
cpio -o -H newc ./* | gzip -n 9 > abc.cgz
```


### cpupower

查看/设置CPU相关信息

```
# cpupower frequency-info //查看P-State相关信息
# cpupower idle-info  //查看C-State相关信息
# cpupower idle-set   //设置C-State相关状态
```


### cp && rm

说明:
* /root/.bashrc里面对cp与rm命令都使用alias进行了重命名
* rm如果遇到冲突的选项，会以最后一个选项为参数执行rm命令。
* cp如果遇到冲突的选项，会以最开始的选项为参数执行cp命令

这样就会导致，在终端下执行cp –f命令时仍然需要手工进行确认。


### dpkg

| 序号|	命令| 说明|
| --- | --- | --- |
| 1 | dpkg -s gcl | 查看安装包的状态 |
| 2 | dpkg -L gcl | 列出某个包所包含的文件 |
| 3 | dpkg –S  /bin/ls | 查看某个文件属于哪个安装包 |
| 4 | dpkg -S \`update-alternatives --display java\` | 当文件是软链接时, 查看某个文件属于哪个安装包|
| 5 | apt-cache search "gcl*" |	查找某个包 |
| 6 | dpkg –l  | grep gcl 	查看某个包是否被安装 |
| 7 | dpkg -i gcl.deb |	安装一个Debian软件包 |
| 8 | dpkg -r gcl.deb |	删除一个已安装的软件包 |
| 9 | dpkg-deb -build |	创建一个新的软件包 |
| 10 | dpkg-deb -info gcl.deb |	从软件包中解压出通用信息 |
| 11 | dpkg-deb -contents gcl.deb | 显示一个软件包的内容 |

相关配置文件：
1）/etc/dpkg/dpkg.cfg 
可以通过 /etc/dpkg/dpkg.cfg 控制 dpkg 的配置，还可以通过主目录中的 .dpkg.cfg 文件提供进一步配置。

2）/var/lib/dpkg
dpkg 工具使用文件系统中/var/lib/dpkg树中的许多文件,尤其是/var/lib/dpkg/status文件包含系统上包的状态信息。


### dhparm

说明：查看硬盘相关信息
```
# dhparm  -a  /dev/sda
```


### dmidecode

说明：查看机器型号
```
# dmidecode | grep "Product Name"
```


### ethtool

说明：查看与设置网卡的相关信息
```
# ethtool eth0
# ethtool –s ethN speed 1000 duplex full port tp autoneg on
```
注: 设置网卡1000Mbit/s，全双工，接口方式为tp，自动协调网卡相关参数


### fdisk

shell脚本中使用
```
#!/bin/bash

fdisk /dev/sda <<END

m

p

END
```


### find

mtime查询的是文件last modified时间，其中最让人迷惑的就是参数 +N 、 N 、 -N 三个参数的意义了。

| 参数 | 说明|
| ---  | --- |
| +N   | -∞ — （当前时间-（N+1）*24）|
| -N   | 当前时间-N*24）— +∞ |
| N	   | 当前时间-(n+1)*24 — （当前时间 - n*24) |

* -mtime 0：24小时内
* -mtime +0： 24小时前 


### hash

说明：缓存command的路径

```
# hash -l  //list所缓存的路径
# hash -r  //清空缓存
```


### ipmitool

说明：查看系统电力消耗
```
# ipmitool sensor
```

### grep
```
# grep  . ./test.txt /dev/null >2/dev/null （追加多个检索文件时会打印出文件名）
```


### growisofs

说明：制作iso的DVD盘
```
# growisofs -dvd-compat -Z /dev/dvdrw1=/mnt/fedora.iso
```
注：/dev/dvdrw1 为/dev目录下dvd设备名称，具体名称要到/dev目录下去查看。


### gzip

解压缩：
```
# gzip abc.cgz -dc | cpio -idumv
```


### ipcs

说明：共享内存查看
```
# ipcs –ma
```


### losetup

* 查看img内容

```
# losetup  –f  xxx.img                （把img文件虚拟成硬盘）
# losetup  –a                         （查看）
# kpartx   –a  /dev/loopX             （映射分区，通过loseup –a查看获得）
# mount  /dev/mapper/loopXpX  /mnt

# umount  /mnt
# kpartx  –d  /dev/loopX
# loseup  –d  /dev/loopX
```

注：
1)当出现vdi等格式的img文件时可能无法kpartx，可以使用qemu-img convert命令把起格式转换为raw格式,然后再尝试。
2)当img文件里面使用的是LVM方式的磁盘分区时，mount时可能出现错误："mount: unknown filesystem  type LVM2_member",
这个时候需要这样挂载：

```
# pvscan                    (查看逻辑卷组)
# vgchange –ay $vol_group   (激活逻辑卷组)
# mount /dev/$vol_group/xxx /mnt
```

* iso文件虚拟成光驱

```
# ln -s /dev/loopN /dev/cdrom
# losetup /dev/loopN  ***.iso
# mount /dev/cdrom  /mnt/cdrom

# umount /mnt/cdrom
# loseup –d /dev/loop7
```


### lsblk

说明：
```
# lsblk
```

查看磁盘是HDD还是SSD
```
# lsblk -d -o name,rota /dev/sda
# cat /sys/block/sda/queue/rotational
```


### lscpu

说明：查看CPU相关信息

```
# lscpu
```
配置文件：cat /sys/device/system/cpu/cpu0/cache/index

```

查看CPU个数：
```
# nproc
# getconf _NPROCESSORS_CONF
# cat /proc/cpuinfo
```


### lshw

查看显卡驱动等信息
```
# lshw –c display
# lspci –vnn
```


### lspci
```
# lspci -vvv
```


### mcpet-kvm

```
# mcpet-kvm -t  <秒数>
```


### mii-tool

说明：查看网卡连接状况
```
# mii-tool –w eth0 eth1 ……
```


### mktemp
说明：创建临时文件供shell脚本使用
```
# mktemp  -dq  /tmp/file. XXXXXXXXXX (必须以X这种格式结尾)
```
注：
1) -d：表示生成一个目录而不是一个文件
2) -q：表示quiet模式（即使出错也不输出一些错误信息到stdout）


### modinfo

查看模块xfs的相关信息
```
# modinfo xfs
```


### mount

说明：远程挂载
```
# mount -t nfs 192.168.2.14:/data/sdb2/RHEL_GA/RHEL7.2GA_x86_64/ /mnt/
# mount -o username=share,password=share  //10.167.226.124/share  /mnt
```

注：
* 当使用root账户都无法改正文件权限时，重新挂载文件所在分区，因为挂载的时候可能是以只读方式挂载的。
* 使用如下命令重新挂载:mount /dev/sda3  -o remount

如果下面命令能挂载成功，说明UUID没有错误。
```
mount UUID=bf0c50bb-8b71-45f8-91e2-f197e2f8b7c9 /home/renyl/，如果能挂载成功，说明UUID没有错误。
```


### multipath

查看:
```
# multipath –ll
# dmsetup ls
```

删除:
```
# multipath -F /dev/mapper/xxx
# dmsetup remove
```


### mv

-T选项的用法：

```
# touch fileA
# mkdir dirA
# ln -sf dirA  linkA
# mv fileA linkA  => fileA will be put into dirA
# mv -T fileA linkA => file A will be rename linkA
```

```
# touch fileA
# mkdir dirA
# mv fileA dirA
# mkdir dirB

# mv dirA dirB    => dirA will be put into dirB
# mv -T dirA dirB => rename dirA to dirB, dirB must be a empty directory
```


### netcat

在server端执行
```
# nc -l -k port  # -l参数表示listen，-k表示连续监听，port表示端口号
# nc -l port |   # 显示出来执行送给bash执行
```

在client端执行
```
# nc  ipaddr  port
# echo  "command" | nc ipaddress portnum
```


### ntpdate

说明：同步时间并设置墙上时间
```
# ntpdate cn.pool.ntp.org
# hwclock -w
```


### numactl

说明：查看与控制CPU和Memory的使用
```
# numactl –hardware   （查看numa node）
```


### oprofile

说明： 
* 配置 OProfile 是否监视内核,这是启动前唯一需要的配置选项,其它选项都是可选的。
* 需要装kernel-debug包。

root用户身份执行以下命令：
```
# opcontrol –-vmlinux=/boot/vmlinuz-`uname –r` //设置是否监视内核

# opcontrol  --vmlinux=/usr/lib/debug/lib/modules/`uname -r`/vmlinux
# opcontrol  --no-vmlinux  //不监视内核

# opcontrol  -p/ --separate=none 
# opcontrol  -l/--list-events //查看可监视事件并设置

# opcontrol --event=DTLB_MISSES:6000:0x01:1:1
# opcontrol  -s/--start  //启动

# opcontrol  --init
# ./path/to/mybinary //运行待分析的程序

# opcontrol  -s/--start
# opcontrol –d/--dump //取数据

# ./my-start-run.sh.bak log-host-nopci-full-memory-oprofile-1
# opcontrol –t/--stop //停止

# opcontrol  -d/--dump
# opreport –l ./path/to/mybinary //分析结果

#opcontrol  -t/--stop
```


### parted

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | parted $DISK p | 显示磁盘分区信息 |
| 2 | parted $DISK rm \$number  |删除disk的number号分区 |
| 3 | parted /dev/$disk -s mklabel gpt| 设置磁盘为gpt模式 |
| 4 | parted -s $DISK  mkpart primary xfs 1049KB \$SIZE | 创建新分区 |


### partprobe

通知操作系统分区表发生的变化
```
# /sbin/partprobe  #通知操作系统分区表发生的变化
```


### pfmon

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | pfmon –l | 列出硬件事件名称 |
| 2 | pfmon –i event_name | 查看事件详细信息 |
| 3 | pfmon –e event_name /path/mybinary | 指定硬件事件并运行程序使 |

当程序运行结束后，pfmon会显示该事件的采样次数，如下所示：
```
pfmon –e L2D_MISSES ./test
  1652 L2D_MISSES
```


### perf

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | perf record –g –o result  ./a.out | 采集信息 |
| 2 | perf record -g -p $pid  -o result | 根据PID来采集信息 |
| 3 | perf report  -g  -i result | 查看采集的信息 |
| 4 | perf list | 列出系统事件 |
| 5 | perf stat –e “kvm:*” -a command | -a监视所有cpu,kvm:*表示所有的kvm |
| 6 | perf stat -e cache-misses  -p 4625 | 采集进程号为4625的cache-misses数 |
| 7 | perf record -q -ag --realtime=1 -m 256 -- sleep 10 |


### powertop

查看：查看进程电力消耗相关信息
```
# powertop
```

### qume-img

说明：创建一个设备文件
```
# qemu-img create -f raw  myfile.img 8G cache=none
```
注：

* 7系下不要加cache=none
* 使用上面的命令后,ls命令查看该文件的大小,会显示8个GB,但是当你用du命令查看该文件时,你会发现该文件的大小为0KB


### raw

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1	| raw /dev/raw/raw<N> <major> <minor>|通过主从设备号绑定|
| 2 | raw /dev/raw/raw<N> /dev/<blockdev>|通过设备来绑定|
| 3	| raw –qa|查看|
| 4 | raw /dev/raw/rawN 0 0| 指定主从设备号为0来解绑|

注：
* 其中N的范围是0-8191。（raw目录不存在可以创建）
* 执行序号1的命令后，会在/dev/raw下生成一个对应的raw[N]文件，用命令方式绑定裸设备在系统重启后会失效。

IO测定：
* blockio模式：不挂载 直接dd操作那个分区。
* raw模式：先raw /dev/raw/raw1   /dev/sdd1 ，再直接操作分区。

相关背景知识：
* 裸设备：也叫裸分区（原始分区），是一种没有经过格式化，不被Linux通过文件系统来读取的特殊字符设备。
  裸设备可以绑定一个分区，也可以绑定一个磁盘。
* 字符设备：对字符设备的读写不需要通过OS的buffer。它不可被文件系统mount。
* 块设备：对块设备的读写需要通过OS的buffer，它可以被mount到文件系统中。


### rpm

说明：强制安装rpm包不检查依赖性
```
# rpm -ivh /home/qemu-kvm-0.12.1.2-2.209.el6.x86_64.rpm  --nodeps
```


### rpm2cpio

说明：解压与查看rpm包内容(rpm包括是使用cpio格式打包的，因此可以先转成cpio然后解压）
```
# rpm2cpio xxx.rpm | cpio –div
```


### seq

说明：以单位1进行递增
```
# for i  in `seq num1 num2`; do echo $i; done
```


### scp
```
# scp  192.168.11.4：/home/renyl/*    ./    (把远程的文件传到本地，可以使用*号)
# scp  localfile   192.168.11.4:/home/renyl (把本地的文件传到远程)
```


### screen

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1	| screen –S  session_name | -S选项表示给会话起个名字|
| 2	| screen -ls | 查看 |
| 3 | screen –r xxx	| 恢复 |
| 4	| ctrl+a+d | detach |
| 5 | exit | 退出|


### ssh

* 在远程机器上运行命令

```
# ssh  root@192.168.122.92 (之间要有空格) “cd /home/renyl/; ./start_run.sh”
```
注: ssh IP “command”(command接在ip之后，同一行)

* 端口转发

```
# ssh -g -L  [bind_address:]port:host:hostport <hostname>
```
说明：
1）表示访问IP地址为bind_address、端口号为port的连接会被SSH转发到IP地址为host、端口号为hostport的应用。
2）SSH 端口转发是通过 SSH 连接建立起来的，我们必须保持这个 SSH 连接以使端口转发保持生效。
   一旦关闭了此连接，相应的端口转发也会随之关闭。
3)我们只能在建立 SSH 连接的同时创建端口转发，而不能给一个已经存在的 SSH 连接增加端口转发

Ex：(访问193.168.246.254:80端口会被转发到193.168.197.232:80上)
```
ssh -g -L 193.168.246.254:80:193.168.197.232:80  193.168.197.232
```

或者在~/.ssh/config文件进行如下配置
```
Host allen-forward

	User renyl
	HostName lucky
	LocalForward localhost:6666 192.168.0.2:22

Host allen
	User renyl
	HostName localhost
	Port 6666
```

说明：
1)首先运行 ssh allen-forward 建立ssh通道
2)然后就可以运行 ssh allen 进行连接了。


### sync

说明：将dirty的内容写回硬盘
```
sync
```

### sysctl

```
sysctl -a #查看系统所有配置参数
sysctl -n kernel.hostname #打印参数值
sudo sysctl -w kernel.hostname="kulong0105" #改变参数值
sudo sysctl -p  #添加设置到/etc/sysctl.conf,使其重启后仍然有效
```


### sar

| 序号 | 命令 | 说明|
| --- | --- | --- |
| 1 | sar  \$interval  $count  -o  sar.log |  采集信息 |
| 2 | sar –f sar.log –P ALL | 采集所有的CPU信息 |
| 3 | sar –f sar.log –P 1,2,3-P | 表示指明具体的CPU |
| 4 | sar –f sar.log –S xx:xx:xx –E xx:xx:xx | 指定起始和结束时间 |
| 5 | f8sar –f sar.log –u ALL –P ALL | 看所有系统信息 |
| 6 | sar –f sar.log –n ALL | 查看网络统计的信息 |


### smartctl

说明：查看磁盘信息
```
# smartctl -i /dev/sda
```


### taskset

说明: 绑定程序到某一个或某些cpu core上

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | taskset –p mask  pid | 根据PID及掩码绑定|
| 2	| taskset –c numlist command | 根据命令及cpulist绑定|
| 3 | taskset -p –c numlist pid | 根据PID及cpulist绑定|
| 4	| taskset –p  pid | 查看|


### tmux

说明：终端多路复用工具

| 序号| 命令 | 说明 |
| --- | --- | --- |
| 1 | tmux | 打开一个匿名的session |
| 2 | tmux new-session -s kulong0105 | 开个一个命名为“kulong0105”的session |
| 3 | tmux ls | 列出所有的session |
| 4 | ctrl+s+d | detach 这个session |
| 5 | tmux a -t kulong0105 | attch 到命名为kulong0105的session |
| 6 | tmux kill-session –t kulong0105 | 杀死命名为“kulong0105”的session |
| 7 | tmux kill-server | 杀死所有session |
| 8 | ctrl+s+t | 显示钟表，很geek |
| 9 | ctrl+s+c | 创建一个新窗口 |
| 10 | ctrl+s+w| 窗口之间的切换 |
| 11 | ctrl+s+b/v | 创建一个新的窗格 |


相关配置：（文件/etc/.mtux.conf或~/.tmux.conf）
```
unbind C-b
set -g prefix C-s
setw -g mode-keys vi

# split window like vim
# vim's defination of a horizontal/vertical split is revised from tumx's

bind b split-window -h
bind v split-window –v

# move arount panes wiht hjkl, as one would in vim after C-w
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# resize panes like vim
# feel free to change the "1" to however many lines you want to resize by,
# only one at a time can be slow
bind < resize-pane -L 10
bind > resize-pane -R 10
bind - resize-pane -D 10
bind + resize-pane -U 10

# bind : to command-prompt like vim
# this is the default in tmux already
bind : command-prompt
```
注: 为了使配置文件生效，先启动一个tmux，按下ctrl+s+:，然后输入source /root/.tmux.conf,按下Enter键即可。


### tc

* 限制输出流量：

method1: (only use tc)

```
#!/bin/bash

export MY_CARD="enp2s0"
tc qdisc add dev $MY_CARD root handle 1: htb
tc class add dev $MY_CARD  parent 1: classid 1:1 htb rate 400mbit
# 同时限制端口速度
tc filter add dev $MY_CARD parent 1: prio 1 protocol ip u32 match ip dst 0.0.0.0/0 match ip sport 80 0xffff flowid 1:1
```

method: (tc with cgroup)

```
#!/bin/bash

export MY_CARD="enp2s0"
echo 0x100001 >  /sys/fs/cgroup/net_cls/net_cls.classid
tc qdisc add dev $MY_CARD root handle 10: htb
tc class add dev $MY_CARD parent 10: classid 10:1 htb rate 400mbit
tc filter add dev $MY_CARD  parent 10: protocol ip prio 10 handle 1: cgroup
```

* 取消输出流量限制：

```
#!/bin/bash

export MY_CARD="enp2s0"
tc qdisc delete dev $MY_CARD root
```

注意:(网络带宽为1000mbit)

a)当设置输出带宽限制为20mbit时：对输入带宽没有任何限制，即输入带宽仍可以达到1000mbit。
b)当设置输出带宽限制为200kbit时，同时还会对输入流量产生限制作用，经测定此时输入带宽只能达到20mbit/s。（不理解）


* 限制输入与输出流量：

```
#!/bin/bash

export MY_CARD1="enp2s0"
export MY_CARD2="ifb0"

modprobe ifb  numifbs=1
ip link set dev $MY_CARD2 up
tc qdisc add dev $MY_CARD1 handle ffff: ingress
tc filter add dev $MY_CARD1  parent ffff: protocol ip u32 match u32 0 0 action mirred egress redirect dev $MY_CARD2

#限制输出
tc qdisc add dev $MY_CARD1 root handle 1: htb default 10
tc class add dev $MY_CARD1 parent 1: classid 1:1 htb rate 50mbit ceil 50mbit burst 1m
tc class add dev $MY_CARD1 parent 1:1 classid 1:10 htb rate 50mbit

#限制输入
tc qdisc add dev $MY_CARD2 root handle 1: htb default 10
tc class add dev $MY_CARD2 parent 1: classid 1:1 htb rate 40mbit ceil 40mbit burst 1m
tc class add dev $MY_CARD2 parent 1:1 classid 1:10 htb rate 40mbit
```

* 取消输入流量限制：

```
#!/bin/bash

export MY_CARD1="enp2s0"
export MY_CARD2="ifb0"

tc qdisc delete dev $MY_CARD1 root
tc qdisc delete dev $MY_CARD2 root
tc qdisc delete dev $MY_CARD1 handle ffff: ingress
```


### yum

使用createrepo配置
```
# mount –o loop  XX.iso  /root/
# cd /root/Packages
# rpm -ivh createrepo-0.9.8-5.el6.noarch.rpm
# cd /root
# createrepo /root/       (表明把仓库文件放在该目录下)
# cd /etc/yum.repos.d
# cat /etc/yum.repos.d/mycdrom.repo
[Base]
name=RHEL7 ISO Base
baseurl=file:///mnt/         #mnt为mount的目录
enabled=1
gpgcheck=0
[root@localhost /]
```

直接使用/etc/ yum.repos.d/file.repo 文件配置：
```
# cd /etc/yum.repos.d/
# cp file.repo file.repo.bak
# cat file.repo
[file]  #最好和文件名保持一致
name=test  #这个随便写
baseurl=file:///mountdir   #这个目录很重要，跟iso文件下的repodata同一层目录
enabled=1
gpgcheck=0
```

yum相关命令

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1	| yum list | 列出所有软件包 |
| 2 | yum list available | 列出可用的包 |
| 3	| yum list updates | 列出可更新的包 |
| 4 | yum install | 安装 |
| 5 | yum remove | 删除 |
| 6 | yum grouplist | 列出可安装的组 |
| 7	| yum groupinstall | 用系统中的组安装 |
| 8 | yum groupinfo | 系统中组的信息 |
| 9 | yum search command | 查找command这个命令在哪个包 |
| 10| yum clean all | 清空缓存 |


### virsh

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | virsh list --all | 查看系统所有虚拟机|
| 2 | virsh list |查看已启动虚拟机|
| 3 | virsh start domain | 启动某个虚拟机|
| 4 | virsh edit domain | 编辑配置文件|
| 5 | virsh dumpxml domain  > tmp.xml | dump配置文件|
| 6 | virsh vcpupin domain \$guest_cpu $host_cpu | 绑定Guest的CPU |
| 7 | virsh vcpuinfo domain | 查看信息|
| 8 | virsh destroy domain | 关闭虚拟机|
| 9 | virsh –c lxc:/// define domain.xml | 定义xml文件|
| 10 | virsh –c lxc:/// start domain | 启动一个lxc |
| 11 | virsh –c lxc:/// console domain | 连接一个lxc |
| 12 | virsh -c qemu+ssh://IP/system list | 查看远程机器的Guest |


### virt-what

说明：检测当前是否在虚拟机环境下
```
# virt-what
```

或者
```
# cat /proc/cpuinfo | grep hypervisor
```


### watch

说明：查看变化信息
```
# watch -d -n1 du -m file1      //-d表示显示后面命令执行的变化
```


### wget

* 在A机器上（IP地址为：193.168.116.53)执行如下命令，从fedora官网拉openstack仓库。

juno:
```
# wget -m -np -p -k -L -P . https://repos.fedorapeople.org/repos/openstack/openstack-juno/epel-7/
# wget -m -np -p -k -L -P . https://repos.fedorapeople.org/repos/openstack/openstack-juno/epel-7/epel/
# wget -m -np -p -k -L -P . https://repos.fedorapeople.org/repos/openstack/openstack-juno/epel-7/repodata/
```
注：-c选项表示断点续,在实际应用中,加了-c选项,网络断开后,重新连接网络后,再执行如上命令,不会继续下载。

* 在A机器上，启动httpd服务。（在/etc/httpd/conf/httpd.conf文件中配置Documentroot参数）

* 在B、C、D机器上，添加yum源，内容如下:
```
[juno_repo]
name=juno
baseurl = http://193.168.116.53/epel-7
enabled=1
gpgcheck=0
```

### xargs

说明：
* 使用find命令的-exec选项处理匹配到的文件时，find命令将所有匹配到的文件一起传递给exec执行。
* 但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。
  错误信息通常是“参数列太长”或“参数列溢出”。
* xargs命令每次只获取一部分文件而不是全部。当find命令把匹配到的文件传递给xargs命令，
  它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。

注：-exec： find命令对匹配的文件执行给出的shell命令（如：find . -type f -exec ls -l { }  \;）

```
# find ./ | xargs dos2unix {}        --- {} 表示所有find到的文件
# ls | xargs -i mv {} {}.bak         --- -i 选项告诉 xargs 用每项的名称替换{}
# find -name ".svn" |xargs rm -rf
```

### xfs

备份与恢复：
```
# xfsdump -f /mountdir/backup  -L lable  -M lable /dev/sda3
# xfsrestore -f /mountdir/backup  ./new_dir
```

配额控制：
```
# mount -o usrquota,grpquota  /dev/xxx  /mountdir
# xfs_quota -x -c 'limit bhard=500m someuser' /mountdir
# xfs_quota -x -c "report"
# repquota -ug /mountdir
```

拷贝：
```
# xfs_copy /dev/sdc1 /dev/sdc2
```

注：
1)使用xfs_copy可以拷贝XFS文件系统，用户可以将一个或几个文件系统拷贝到磁盘分区或文件中。
2)如果多个目标，可并行。
3)目标文件直接被格式化xfs文件系统
4)如果原始文件系统小于新文件系统，建议使用mkfs.xfs/xfsdump/xfsrestore


冷冻与恢复：
```
# xfs_freeze -f /mountdir    //暂停
# xfs_freeze -u /mountdir    //恢复
```
注：源文件系统应该被卸载或者只读被挂载或者被freeze，目标文件系系统应该被卸载。


修复：
```
# xfs check /dev/sdc1
```
注：xfs文件系统的检测和修复在运行xfs_check 、xfs_repair之前，被检测的文件系统应该被卸载或者只读被挂载，否则文件系统会崩溃。

