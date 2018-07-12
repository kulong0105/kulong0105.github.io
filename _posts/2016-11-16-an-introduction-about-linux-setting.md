---
title: Linux常用设置
category: linux
tags:
- linux
- performance
---

## 介绍

本文将对Linux常用的一些设置进行介绍。

### cgroup设置

利用cgroup对系统的 CPU / MEM / IO 等资源进行监控与限制

1) 可用子系统

| 序号|	子系统|	说明|
| --- | --- | --- | 
| 1	| blkio	| 为块设备设定输入/输出限制（如，物理设备）|
| 2	| cpu	| 使用调度程序提供对CPU的cgroup任务访问。(控制CPU的利用率)| 
| 3	| cpuacct | 自动生成cgroup中任务所使用的CPU报告 | 
| 4	| cpuset | 为cgroup中的任务分配独立CPU和内存节点| 
| 5	| devices | 允许或者拒绝cgroup中的任务访问设备 |
| 6	| freezer | 挂起或者恢复cgroup中的任务 | 
| 7	| memory | 设定cgroup中的任务使用内存限制 | 
| 8	| net_cls | 使用等级识别符（classid）标记网络数据包，允许Linux流量控制程序（tc）识别从具体cgroup中生成的数据包|

<!--more-->

2) 挂载方式
```
# mount –t cgroup –o cpuset none /cgroup/cpuset
```

3) 查看cgroup资源

```
# cat /proc/cgroup
# cat /proc/$pid/cgroup
```

4) 使用说明

当启动新进程时，它会继承其父进程的组群。因此在cgroup中启动进程的方法是将shell进程移动到那个组群中，然后在那个shell中启动该进程。

```
# mkdir /cgroup/label/mytest
# echo $$ >/cgroup/$label/mytest/tasks
# run command
```

注：
* lable可为上面子系统的标志，如cpuset、cpu、memory等。
* 有些子系统拥有强制参数，在可以将任务移动到使用那些子系统的cgroup前必须设定这些参数,
  如：在将任务移动到使用 cpuset子系统的cgroup前，必须为那个cgroup定cpuset.cpus和cpuset.mems 参数。
* 只能为该root组群设定少量参数。因为root组群拥有所有现有资源，因此定义某些参数限制现有进程就没有任何意义，例如cpuset.cpu 参数。


由于command执行后，现有shell中仍在cgroup中。因此更好的方法应为：

```
# bash -c "echo \$$ > /cgroup/mytest/tasks && command"
```

5) 对CPU资源进行限制

```
# mkdir  /cgroup/cpuset/renyl       #创建一个新的组
# mkdir  /cgroup/cpu/renyl  
# echo 0 >/cgroup/cpu/renyl/cpuset.cpus   #cpus和mems必须要设置
# echo 0 >/cgroup/cpu/renyl/cpuset.mems    
# cat  /cgroup/cpu/renyl/cpu.cfs_period_us
100000
# echo 50000 >/cgroup/cpu/renyl/cpu.cfs_quota_us  #控制cpu利用率为50%
# echo $pid > /cgroup/cpuset/renyl/tasks
# echo $pid > /cgroup/cpu/renyl/tasks
```

6) 对Memory资源进行限制

```
# mkdir /cgroup/memory/renyl/
# echo 10M > /cgroup/memory/renyl/memory.limit_in_bytes
# echo $$ > /cgroup/memory/renyl/tasks
```

7) 对IO资源进行限制：

```
# mkdir /cgroup/blkio/renyl/
# ls -al /dev/sda                          
brw-rw----. 1 root disk 8, 0 Mar 23 13:41 /dev/sda
# echo "8:0 1000" > /cgroup/blkio/renyl/blkio.throttle.read_bps_device  //单位为bytes
# echo "8:0 1000" > /cgroup/blkio/renyl/blkio.throttle.write_bps_device
# echo "8:0 0" > /cgroup/blkio/renyl/blkio.throttle.read_bps_device     //解除
```


### drop_caches设置

清空相关cache

```
# echo 1 > /proc/sys/vm/drop_caches  #To free pagecache
# echo 2 > /proc/sys/vm/drop_caches  #To free dentries and inodes
# echo 3 > /proc/sys/vm/drop_caches  #To free pagecache, dentries and inodes
# echo 0 > /proc/sys/vm/drop_caches (默认情况下)
```

注：
* 由于该文件是nondestructive operation and dirty objects are not freeable，因此可以先使用sync同步一下。
* 使用上述不能确保能够清空相关cache，使用umount相关分区可以确保清空相关cache。


Buffer Cache 和Page Cache的区别:
* Buffer Cache(Slab cache):用来缓存文件的（dentries and inodes等等）
* Page Cache:完完全全是缓存文件的内容


### Selinux设置

查看：
```
# getenforce
# sestatus
```

设置：
```
# setenforce 1
# vim /etc/selinux/config	   //重启后生效
```


### CPU和MEM设置 

CPU设置:
```
echo 0 > /sys/devices/system/cpu/cpuN/online //禁止该cpu的运行
vim /boot/grub/grub.conf 增加 isolcpus=6-11  //指示系统默认不使用6-11 CPU,这样在系统启动后就可以把测试进程绑定至保留的CPU
```

MEM设置:
```
vim  /boot/grub/grub.conf 增加MEM=8GB     //限制系统只能使用8GB的内存
```


### dirty_cache设置

| 序号 | 文件 | 说明 |
| --- | --- | --- |
| 1 | /proc/sys/vm/dirty_background_ratio| 表示脏数据到达系统整体内存的百分比时触发pdflush进程把脏数据写回磁盘。(非阻塞方式) | 
| 2	| /proc/sys/vm/dirty_expire_centisecs| 表示脏数据在内存中驻留时间超过该值时触发pdflush进程在下一次将把这些数据写回磁盘 |
| 3	| /proc/sys/vm/dirty_ratio| 表示脏数据到达系统整体内存的百分比时触发pdflush进程把脏数据写回磁盘。（阻塞方式）|
| 4	| /proc/sys/vm/dirty_writeback_centisecs| 表示pdflush进程周期性间隔多久把脏数据写回磁盘 |


回写dirty页的方式：
* 当脏页时间超过参数/proc/sys/vm/dirty_expire_centisecs时触发pdfulsh回写脏页,这种方式启动的回写线程只回写超时的dirty页，
  不会回写没超时的脏页。
* 当脏页占系统内存比例超过参数/proc/sys/vm/dirty_background_ratio 时,系统会唤醒pdflush线程回写脏页直到比例低于
  dirty_background_ratio,这种方式不会导致write系统调用被阻塞。
* 当脏页占系统内存的比例超/proc/sys/vm/dirty_ratio的时候,系统会调用pdfulsh线程写脏页直到脏页比例低于
  /proc/sys/vm/dirty_ratio。这种方式会阻塞write系统调用。


### hugepage设置

1) hugepage原理的相关说明：
* 系统进程是通过虚拟地址访问内存,但CPU是通过物理地址访问内存的,为了提高访存速度,CPU的组件TLB会缓存最近的虚拟地址到物理地址的映射。
* 当使用hugepage时,由于TLB中每一个表项可以映射更大的物理地址空间,对于使用大量内存的程序,可以减少进程页表的项数,
  从而降低TLB miss rate,节约寻址时间，提高了性用程序的性能
* hugepage成功分配的前提是系统中存在大量连续的物理内存，否则分配hugepage失败。
* 使用hugepages的内存页是不能被swap，也不能被其它使用普通页面的进程使用，永远常驻在内存中。因此，会减少内存页交换的额外开销。

注：TLB以CPU Core为资源单位，所有运行于CPU Core的线程和进程使用同一个TLB。只要有执行一个context switch TLB就会被刷新。

2) hugepage大小的相关说明：
* 可以通过 *cat /pro/meminfo | grep Hugepagesize* 来查看。一般是2048KB 。
* 可以通过命令*getconf PAGE_SIZE*或函数*sysconf(PAGESIZE)*函数查看普通的的内存页大小，一般是4096Byte。


3) hugepage的设置说明： 

| 序号 | 方法 | 说明 |
| --- | --- | --- |
| 1	| sysctl vm.nr_hugepages=N |重启后失效 |
| 2	| echo N > /proc/sys/vm_nr_hugepages|重启后失效|
| 3	| /etc/sysctl.conf下添加vm.nr_hugepages=N |	sysctl –p 使其生效,重启后仍然有效 |
| 4	| grub命令行中添加hugepages=N |	无 |

注：hugetlbfs文件系统： mount -t hugetlbfs hugetlbfs /dev/hugepages

4) Guest使用hugepage的说明： 
* 在Host开启hugepage
* 在qemu启动时加上参数，示例：qemu-system-x86_64 -m 2048 -hda /mnt/rhel6.img -mem-path /dev/hugepages
* 如果是用libvirt来启动KVM的，需要在Guest的XML配置文件添加如下的参数： 

```
<memoryBacking>

<hugepages/>

</memoryBacking>
```
注：
* 设置了多少的hugepage，free命令就会显示多少内存被使用。
* 启动Guest后，可以看到Host上的HugePages_Free（使用cat /pro/meminfo查看）数量有所减少。


5) 关于transparent_hugepage的相关说明：
* Linux 操作系统采用了基于hugetlbfs特殊文件系统2M字节大页面支持,使得应用程序可以根据需要灵活地选择虚存页面大小。
* 透明大页面使得系统能够自动创建、管理和使用大页面,不需要修改程序代码。
* THP目前只支持匿名映射如堆和栈空间。
* 配置文件为/sys/kernel/mm/redhat_transparent_hugepage/enabled
* 在程序调用的时候有时候分配一个大页面,然后它会在后台自动为你拼装,而且内存不够的时候还可以把大页劈开进行交换。
* 透明大页面的个数可以通过/proc/meminfo的参数AnonHugePages来查看。


### 网络相关设置

1) 网卡配置文件说明：

| 序号 | 参数 | 说明 |
| --- | --- | --- |
| 1 | DEVICE=eth0 | 指定网卡设备名称为eth0 |
| 2 | HWADDR=00:0c:29:f9:67:5b | 指定物理网卡地址为00:0c:29:f9:67:5b |
| 3 | NM_CONTROLLED=yes	| 设备eth0是否可以由Network Manager图形管理工具托管 |
| 4 | ONBOOT=no  | 启动网络服务是否使eth0设备生效 |
| 5 | BOOTPROTO=dhcp | dhcp方式获取IP地址 |
| 6 | TYPE=Ethernet	| 网络类型为以太网模式 |
| 7 | BROADCAST=192.168.10.255 | 设置广播地址 |
| 8 | IPADDR=192.168.10.238	| 设置静态IP地址 |
| 9 | NETMASK=255.255.255.0	| 设置子网掩码 |
| 10 | GATEWAY=192.168.10.12 | 设置网关地址 |


2) 对网关进行相关配置：
```
# route  add  -net 169.254.0.0  netmask  255.255.0.0  dev  eth1  （添加一个网关）
# route  del  -net 169.254.0.0  netmask  255.255.0.0  dev  eth1  （删除一个网关）
```


### KSM设置

关于KSM(Kernel Samepage Merging, Kernel Shared Memory)服务的说明如下：
* 默认情况下，当内存使用超过80%时，ksmtuned会启动KSM服务，对相同的页面进行合并，减少内存使用。
* 可以通过修改文件/etc/ksmtuned.conf里的参数KSM_THRES_COEF参数改变什么时候启动KSM服务。
  如，KSM_THRES_COEF=20表示内存不足20%时启动KSM服务，可设置范围为0%-100%。


### clocksource设置 

```
# cat /sys/devices/system/clocksource/clocksource0/available_clocksource 
tsc hpet acpi_pm 
# cat /sys/devices/system/clocksource/clocksource0/current_clocksource 
tsc
# 
```


### cstate设置

```
# cat /sys/module/intel_idle/parameters/max_cstate  //查看
# cpupower idle-set  //设置
```

其他设置方法：
1) BIOS设置: Package C-State
2) GRUB设置: idle=poll、mwait、halt、nomwait
	* processor.max_cstate
	* intel_idle.max_cstate

3) cpu_dma_latency	/dev/cpu_dma_latency

注：
* intel_idle.max_cstate=0将禁用intel_idle驱动，使用acpi_idle驱动。
* cpu_dma_latency表示进入到C0状态的最大唤醒时间。若该值为0，表示阻止进入睡眠状态。


P-State的相关设置：（RHEL6默认的驱动为acpi_cpufreq，RHEL7系默认的驱动为intel_pstate）
1)Governor驱动设置：在Grub命令行中添加intel_pstate=disble即可，使用cpupower frequency-info进行确认。
2)Governor设置：通过/sys/devices/system/cpu/cpuN/cpufreq/scaling_governor设置。

