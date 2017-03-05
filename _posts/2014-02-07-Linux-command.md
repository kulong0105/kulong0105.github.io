---
title: Linux command
category: linux
tags:
- linux 
- command
---


## Linux command

### 1. virsh

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | virsh list --all | 查看系统所有虚拟机|
| 2 | virsh list |查看已启动虚拟机|
| 3 | virsh start domain | 启动某个虚拟机|
| 4 | virsh edit domain | 编辑配置文件|
| 5 | virsh dumpxml domain  > tmp.xml | dump配置文件|
| 6 | virsh vcpupin domain $guest_cpu $host_cpu | 绑定Guest的CPU |
| 7 | virsh vcpuinfo domain | 查看信息|
| 8 | virsh destroy domain | 关闭虚拟机|
| 9 | virsh –c lxc:/// define domain.xml | 定义xml文件|
| 10 | virsh –c lxc:/// start domain | 启动一个lxc |
| 11 | virsh –c lxc:/// console domain | 连接一个lxc |
| 12 | virsh -c qemu+ssh://IP/system list | 查看远程机器的Guest |
      

### 2. parted 

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | parted $DISK p | 显示磁盘分区信息 | 
| 2 | parted $DISK rm $number  |删除disk的number号分区 |
| 3 | parted /dev/$disk -s mklabel gpt| 设置磁盘为gpt模式 | 
| 4 | parted -s $DISK  mkpart primary xfs 1049KB $SIZE | 创建新分区 | 


### 3. perf

| 序号 | 命令| 说明|
| --- | --- | --- |
| 1 | perf record –g –o result  ./a.out | 采集信息 |
| 2 | perf record -g -p $pid  -o result | 根据PID来采集信息 |
| 3 | perf report  -g  -i result | 查看采集的信息 |
| 4 | perf list | 列出系统事件 |
| 5 | perf stat –e “kvm:*” -a command | -a监视所有cpu,kvm:*表示所有的kvm |
| 6 | perf stat -e cache-misses  -p 4625 | 采集进程号为4625的cache-misses数 |
| 7 | perf record -q -ag --realtime=1 -m 256 -- sleep 10 | 
| 8 | perf report --header -U -g fractal,5 > perf-report | 


### 4. pfmon
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


### 5. oprofile 
说明： 
1. 配置 OProfile 是否监视内核,这是启动前唯一需要的配置选项,其它选项都是可选的。
2. 需要装kernel-debug包。

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
