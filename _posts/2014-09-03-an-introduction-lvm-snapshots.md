---
title: LVM Snapshot简介
category: linux
tags:
- lvm
- snapshot
---

## 介绍

1. Logical Volume Manager (LVM)提供了对任意一个Logical Volume(LV)做“快照”(snapshot)的功能，
   以此来获得一个分区一致性备份。

2. LVM的snapshot是通过“写时复制”(copy on write) 的方法实现：
   *  当一个snapshot创建的时候，仅拷贝原始卷里数据的元数据(meta-data)。创建的时候，
      并不会有数据的物理拷贝，因此snapshot的创建几乎是实时的。
   *  当原始卷上有写操作执行时，snapshot跟踪原始卷块的改变，这个时候原始卷上将要改
      变的数据在改变之前被拷贝到snapshot预留的空间里，因此这个原理的实现叫做写时复制(copy-on-write)。

3. 在写操作写入块之前，CoW会将原始数据移动到snapshot空间里，这样就保证了所有的数据在
   snapshot创建时保持一致。而对于snapshot的读操作，如果是读取数据块是没有修改过的，
   那么会将读操作直接重定向到原始卷上，如果是要读取已经修改过的块，那么就读取拷贝到snapshot中的块。

<!--more-->

## 快速使用

### 创建逻辑卷
```
# pvcreate /dev/sda1 /dev/sdb1
# vgcreate -s 32M  vg_data /dev/sda1 /dev/sdb1
# lvcreate -L 100G -n lv_data vg_data
```
注： -s用来设置PE大小


### 扩容
```
# pvcreate /dev/sdc1
# vgextend vg_data /dev/sdc1
# lvextend -L +50G /dev/vg_data/lv_data
# xfs_growfs $MOUNT_POINT
```
注： 文件系统不需要umount


### 缩容
```
# umount /dev/vg_data/lv_data
# lvreduce -L 20G -r /dev/vg_data/lv_data
No modify flag set, skipping filesystem flush and exiting.
fsadm: Xfs filesystem shrinking is unsupported
  fsadm failed: 1
  Filesystem resize failed.
 #
```
注：   
- 文件系统需要umount
- xfs文件系统不支持容缩:[参考](http://xfs.org/index.php/XFS_FAQ#Q:_Is_there_a_way_to_make_a_XFS_filesystem_larger_or_smaller.3F)


### 创建snaphost与恢复
```
# lvcreate -L 30GB -s -n lv_data_snapshot /dev/vg_data/lv_data
# umount /dev/vg_data/lv_data
# lvconvert --merge /dev/vg_data/lv_data_snapshot
```
注：  
- 恢复snapshot时，需要umount原逻辑卷
- 可以通过修改 /etc/lvm/lvm.conf 的 snapshot_autoextend_threshold 字段来自动扩容snaphost逻辑卷的大小 (不需要重启任何服务）
- lvm snapshot在新增文件时会引起数据块被copy到snapshot上，两个原因：
    - a): lvm snapshot 跟文件系统无关，所以当新增加一个文件时，它是无感的，不知道是add还是update
    - b): 由于是对文件系统无感，所以就是通过block来感知的，所有数据都是以block方式copy
- lvm thinpool snapshot不会修改任何原始数据，会创建软链接指向所有原始数据, 所有操作都是直接在snapshot上操作
- lvm thinpool snapshot在新增文件的情况下，就不会引起原始数据块copy


### 创建thinpool及thinpool逻辑卷
```
# pvcreate  /dev/sda1 /dev/sdb1
# vgcreate -s 32M vg_data_thin /dev/sda1 /dev/sdb1
# lvcreate -L 100GB --thinpool lv_data_thinpool vg_data_thin
# lvcreate -V 10GB --thin -n lv_data_thinpool_client1  vg_data_thin/lv_data_thinpool
# lvcreate -V 20GB --thin -n lv_data_thinpool_client2  vg_data_thin/lv_data_thinpool
# lvextend -L +100GB /dev/vg_data_thin/lv_data_thinpool
# lvextend -L +10GB /dev/vg_data_thinpool/lv_data_thinpool_client1
```
注：  
- 不能对thinpool进行缩容
- 可以对thinpool以及thinpool_client进行扩容
- 可以通过修改 /etc/lvm/lvm.conf 的thin_pool_autoextend_threshold 字段来自动扩容thin_pool逻辑卷的大小(不需要重启任何服务）


### 条带化
```
# pvcreate /dev/sda1 /dev/sdb1 /dev/sdc1/ dev/sdd1
# vgreate -s 16M  vg_strip /dev/sda1 /dev/sdb1/ /dev/sdc1 /dev/sdd1
# lvcreate -L 100GB -i4 -n lv_strip1 vg_strip
# lvcreate -L 100GB -i3 -I 256 -n lv_strip2 vg_strip
```
注：
- 可以使用`lvdisplay /dev/vg_strip/lv_strip1 -m`查看
-  -i 设置stripes
-  -I 设置stripe_size


调查发现, 设置lvm的“条带化”功能时，Blocksize的不同对性能影响很大（默认的stripsizes大小为64KB）

| type | bs | throughout |
| --- | --- | --- |
| linear | 1K | 17.8 MB/s |
| strip |  1K | 13.9 MB/s|
| linear | 4K|72.3 MB/s |
| strip | 4K| 44.0 MB/s|
| linear | 16K| 80.5 MB/s|
| strip | 16K| 98.7 MB/s|
| linear | 32K|135 MB/s |
| strip | 32K|142 MB/s |
| linear | 64K| 209 MB/s|
| strip | 64K|209 MB/s |
| linear | 128K| 286 MB/s|
| strip | 128K| 618 MB/s|
| linear | 256K| 356 MB/s|
| strip | 256K| 696 MB/s|
| linear | 1M| 434 MB/s|
| strip | 1M|709 MB/s |

可以得出：
- 在bs小于4K时，linear性能更好
- 在bs 4K ～ 64时，linear 和 strip 性能基本一致
- 在bs大于128时，strip性能明显要好


## 查看与下载

完整文章请转到下面link:
[LVM Snapshot简介](https://github.com/kulong0105/kulong0105.github.io/blob/master/documents/LVM_Snapshot%E7%AE%80%E4%BB%8B.pdf)

[下载链接](https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/LVM_Snapshot%E7%AE%80%E4%BB%8B.pdf)


## reference
- [https://linux.cn/article-3218-1.html](https://linux.cn/article-3218-1.html)   
- [Logical Volume Manager Administration](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/)
