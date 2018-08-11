---
title: ceph部署简介
category: linux
tags:
- ceph
---

## 介绍

本文将对ceph部署的基本流程以及相关操作进行介绍。

<!--more-->

## 安装前准备
    - 配置ntp服务
    - 配置无密码访问
    - 修改/etc/hosts使得集群机器之间可以使用短域名直接互相访问
    - 关闭selinux
    - 关闭iptables服务或者开放相应端口号


## 安装部署

- 安装ceph（所有节点）
    ```
    yum install -y ceph
    ```

- 安装ceph-deploy
    ```
    yum install -y ceph-deploy
    ```

- 创建部署目录
    ```
    mkdir -p /root/my-cluster && cd /root/my-cluster
    ```

- 开始部署
    ```
    ceph-deploy new ceph-1 ceph-2 ceph-3
    ```
    - 生成`ceph.conf, ceph-deploy-ceph.log, ceph.mon.keyring`三个文件
    - new后面跟的是部署MON节点的短域名`hostname -s`，需要奇数个MON节点


- 更新配置文件(/root/my-cluster/ceph.conf)
    - public_network = 192.168.50.0/24: 配置外网网络
    - cluster_network = 10.0.10.0/24: 配置内部网络
    - mon_clock_drift_allowed = 2: 配置MON节点之间的时间差在2秒之内
    - osd_journal_size = 20480:  默认为5GB，设置为20GB
    - mds_cache_memory_limit = 4294967296: 默认为1GB，设置为4GB
    - mds_recall_state_timeout = 120: 默认为60s,设置为120s
    - bluestore_cache_size_hdd = 4294967296: 默认为1GB，设置为4GB


- 部署monitor
    ```
    ceph-deploy mon create-initial
    ```

    - ceph-deploy读取配置文件中的mon_initial_members的各个主机，然后依次SSH前往各个主机：
        - 将**部署目录**下的ceph.conf推送到新节点的/etc/ceph/目录下。
        - 创建/var/lib/ceph/mon/$cluster-$hostname/目录。
        - 检查MON目录下是否有done文件，如果有则直接跳到第6步。
        - 将ceph.mon.keyring拷贝到新节点，并利用该秘钥在MON目录下建立MON数据库。
        - 在MON目录下建立done文件，防止重新建立MON。
        - 启动MON进程。
        - 查看 `/var/run/ceph/$cluster-mon.$hostname.asok` SOCKET文件，这个是由MON进程启动后生成的，输出MON状态。
    - 在所有的MON都建立好后，再次前往各个主机，查看所有主机是否运行并且到达法定人群(quorum)。如果有没到到的，直接结束报错。如果都到达了，执行下一步。
    - 调用auth get-or-create方法创建(如果不存在)或者拉取(已经存在)MON节点上的以下几个keyring到部署目录中：
         ```
        ceph.bootstrap-mds.keyring
        ceph.bootstrap-osd.keyring
        ceph.bootstrap-rgw.keyring
        ceph.client.admin.keyring
        ```

- 推送配置文件和key
    ```
    ceph-deploy admin  ceph-1 ceph-2 ceph-3
    ```

- 部署mgr
    ```
    ceph-deploy mgr create ceph-1 ceph-2 ceph-3
    ```

- 部署osd
    ```
    ceph-deploy disk zap ceph-1 /dev/vdb
    ceph-deploy osd create ceph-1 --data /dev/sdb --block-db /dev/sdc

    ceph-deploy disk zap ceph-2:sdb
    ceph-deploy osd create ceph-2 --data /dev/sdb --block-db /dev/sdc

    ceph-deploy disk zap ceph-3:sdb
    ceph-deploy osd create ceph-3 --data /dev/sdb --block-db /dev/sdc
    ```

    注： zap disk可能会失败，尝试先删除vg/lv/pv，再wipefs

- 部署mds
    ```
    ceph-deploy mds create ceph-1 ceph-2 ceph-3
    ```

- 检查状态
    ```
    ceph -s
    ```

- 增加rbd池
    ```
    ceph osd pool create rbd  # pg 和 pgp 默认为100
    ceph osd pool set rbd pg_num 128
    ceph osd pool set rbd pgp_num 128
    ```

- 管理ceph服务
    - 启动所有服务
    ```
    systemctl start ceph.target
    ```

    - 启动所有osd/mon/mds/mgr服务
    ```
    systemctl status ceph-osd.target
    systemctl status ceph-mon.target
    systemctl status ceph-mds.target
    systemctl status ceph-mgr.target
    ```

    - 启动单个osd/mon/mds/mgr服务
    ```
    systemctl start ceph-osd@0
    systemctl start ceph-mon@ceph-1
    systemctl start ceph-mgr@ceph-1
    systemctl start ceph-mds@ceph-1
    ```


- 注意点
    - `ceph-deploy` 命令必须在部署目录(/root/my-cluster)目录下执行
    - 不要直接修改某个节点的/etc/ceph/ceph.conf文件，而是修改部署目录下的配置文件(/root/my-cluster/ceph.conf），然后推送到各节
    `ceph-deploy--overwrite-conf config push ceph-1 ceph-2 ceph-3`


## PG 状态

PG会存在各种状态，下面列出了PG的各种状态和可能的原因：

状态  | 原因
---|---
Creating | 创建pg时
Peering | 在达成统一状态时，但不表示每个副本有最新的内容
Active | 达到了统一状态，可读可写
Clean | 主osd和副本osd都有最新的值
Degraded | 主osd和副本osd存储不同步
Recovering | 当一个osd挂，过了一段时间后，又恢复了
Backfilling | 新增一个osd，或删除一个osd
Remapped| 当pg对应的osd 从 [0,1,2] -> [3,4,5]
Stale | 主osd挂了
Misplaced | 当pg对应的osd 从 [0,1,2] -> [1,2,3]时，osd3还没有完全backfill
Incomplete | 当pg对应的osd 从 [0,1,2] -> [1,2,3]时,osd3还没有完全backfill, osd0/1/2全挂了


## OSD 设置

可以通过`ceph osd set <flat>` 来设置osd的默认状态和行为：

Flag | Description
--- | ---
noin | Prevents OSDs from being treated as in the cluster.
noout | Prevents OSDs from being treated as out of the cluster.
noup | Prevents OSDs from being treated as up and running.
nodown | Prevents OSDs from being treated as down.
full | Makes a cluster appear to have reached its full_ratio, and thereby prevents write operations.
pause | Ceph will stop processing read and write operations, but will not affect OSD in, out, up or down statuses.
nobackfill | Ceph will prevent new backfill operations.
norebalance | Ceph will prevent new rebalancing operations.
norecover | Ceph will prevent new recovery operations.
noscrub | Ceph will prevent new scrubbing operations.
nodeep-scrub | Ceph will prevent new deep scrubbing operations.
notieragent | Ceph will disable the process that is looking for cold/dirty objects to flush and evict.

## 增加/删除节点

#### 增加mon
    - ceph-deploy mon add ceph-4

#### 删除mon
    - ceph-deploy mon destroy ceph-4

#### 增加osd
    - ceph-deploy osd create ceph-1 --data /dev/sdb --block-db /dev/sdc

#### 删除osd
    - systemctl disable ceph-osd@4
    - systemctl stop ceph-osd@4
    - ceph osd purge osd.4 --yes-i-really-mean-it

    或者

    - systemctl disable ceph-osd@4
    - systemctl stop ceph-osd@4
    - ceph osd out 4
    - wait status is “active + clean"
    - ceph osd crush remove osd.4

#### 增加mgr
    - ceph-deploy mgr create ceph-4

#### 删除mgr
    - systemctl disable mgr@ceph-4
    - systemctl stop mgr@ceph-4

#### 增加mds
    - ceph-deploy mds create ceph-4

#### 删除mds
    - systemctl disable mds@ceph-4
    - systemctl stop mds@ceph-4


## 集群卸载

    ceph-deploy purge ceph-1 ceph-2 ceph-3
    ceph-deploy purgedata ceph-1 ceph-2 ceph-3
    ceph-deploy forgetkeys
    rm ceph .*


## RBD使用

    - rbd pool init rbd
    - rbd create rbd/foo --size 5G (默认单位为M)
    - rbd ls
    - rbd info rbd/foo
    - rbd resize --size 10G rbd/foo
    - rbd resize --size 1G rbd/foo --allow-shrink
    - rbd rm rbd/foo
    - rbd map foo
    - ls /dev/rbd/foo


## CephFS使用

    - ceph osd pool create cephfs_data 128
    - ceph osd pool create cephfs_metadata 64
    - ceph fs new myceph cephfs_metadata cephfs_data
    - key_cert=$(grep key ./my-cluster/ceph.client.admin.keyring)
    - echo "$key_cert" | cut -f2- -d"=" | tr -d  " " > /etc/ceph/admin.secret

#### kernel client 挂载
    - mount -t ceph 192.168.60.12:6789:/  -o name=admin,secretfile=/etc/ceph/admin.secret  /mnt

#### fuse 挂载
    - ceph-fuse -k /etc/ceph/ceph.client.admin.keyring -m 192.168.60.12:6789 /mnt


## Quota使用

#### set
    - setfattr -n ceph.quota.max_bytes -v 100000000 /some/dir     # 100 MB
    - setfattr -n ceph.quota.max_files -v 10000 /some/dir         # 10,000 files


#### view

    - getfattr -n ceph.quota.max_bytes /some/dir
    - getfattr -n ceph.quota.max_files /some/dir

#### unset
    - setfattr -n ceph.quota.max_bytes -v 0 /some/dir
    - setfattr -n ceph.quota.max_files -v 0 /some/dir



## 命令行

#### health

- ceph -s --conf /etc/ceph/ceph.conf --name client.admin --keyring /etc/ceph/ceph.client.admin.keyring
- ceph health
- ceph quorum_status --format json-pretty
- ceph osd dump
- ceph osd stat
- ceph mon dump
- ceph mon stat
- ceph mds dump
- ceph mds stat
- ceph pg dump
- ceph pg stat


#### osd/pool

- ceph osd tree
- ceph osd pool ls detail
- ceph osd pool set rbd crush_ruleset 1
- ceph osd pool create sata-pool 256 rule-sata
- ceph osd pool create ssd-pool 256 rule-ssd
- ceph osd pool set data min_size 2


#### config

- ceph daemon osd.0 config show (在osd节点上执行)
- ceph daemon osd.0 config set  mon_allow_pool_delete true（在osd节点执行，重启后失效）
- ceph tell osd.0 config set mon_allow_pool_delete false （任意节点执行，重启后失效）
- ceph config set osd.0 mon_allow_pool_delete true(只支持13.x版本，任意节点执行，重启有效,要求配置选项不在配置配置文件中，否则mon会忽略该设置）


#### log
- ceph log last 100


#### map
- ceph osd map
- ceph pg dump
- ceph pg map x.yz
- ceph pg x.yz query

#### auth

- ceph auth get client.admin --name mon. --keyring /var/lib/ceph/mon/ceph-$hostname/keyring
- ceph auth get osd.0
- ceph auth get mon.
- ceph auth ls


#### crush
- ceph osd crush add-bucket root-sata root
- ceph osd crush add-bucket ceph-1-sata host
- ceph osd crush add-bucket ceph-2-sata host
- ceph osd crush move ceph-1-sata root=root-sata
- ceph osd crush move ceph-2-sata root=root-sata
- ceph osd crush add osd.0 2 host=ceph-1-sata
- ceph osd crush add osd.1 2 host=ceph-1-sata
- ceph osd crush add osd.2 2 host=ceph-2-sata
- ceph osd crush add osd.3 2 host=ceph-2-sata


- ceph osd crush add-bucket root-ssd root
- ceph osd crush add-bucket ceph-1-ssd host
- ceph osd crush add-bucket ceph-2-ssd host


- ceph osd getcrushmap -o /tmp/crush
- crushtool  -d /tmp/crush -o /tmp/crush.txt
- update /tmp/crush.txt
- crushtool -c /tmp/crush.txt  -o /tmp/crush.bin
- ceph osd setcrushmap -i /tmp/crush.bin


## auth

客户端与Ceph 集群进行交互，至少需要知道四条信息:
- 集群的 fsid
- 集群的 Monitor的IP 地址，必须先连上MON之后才能获取集群信息。
- 一个用于登陆的用户名
- 登陆用户对应的密码


## Misc

#### recommends
- RADOS Cluster: Reliable Autonomic Distributed Object Store

- For high availability, you should run a production Ceph cluster with AT LEAST three monitors. Ceph uses the Paxos algorithm, which requires a consensus among the majority of monitors in a quorum. With Paxos, the monitors cannot determine a majority for establishing a quorum with only two monitors. A majority of monitors must be counted as such: 1:1, 2:3, 3:4, 3:5, 4:6, etc.

- When adding a monitor on a host that was not in hosts initially defined with the ceph-deploy new command, a public network statement needs to be added to the ceph.conf file

- Before you can write data to a placement group, it must be in an active state, and it should be in a
clean state

- Red Hat recommends deploying an odd number of Monitors, but it is not mandatory.

- Do not let a cluster reach the full ratio before adding an OSD

- The minimum size of a journal partition is 5 GB. Journal partitions bigger than 10 GB are not usually needed.


#### config file

```
	# By default, Ceph makes 3 replicas of objects. If you want to make four
	# copies of an object the default value--a primary copy and three replica
	# copies--reset the default values as shown in 'osd pool default size'.
	# If you want to allow Ceph to write a lesser number of copies in a degraded
	# state, set 'osd pool default min size' to a number less than the
	# 'osd pool default size' value.

	osd pool default size = 3  # Write an object 3 times.
	osd pool default min size = 2 # Allow writing two copies in a degraded state.

	# Ensure you have a realistic number of placement groups. We recommend
	# approximately 100 per OSD. E.g., total number of OSDs multiplied by 100
	# divided by the number of replicas (i.e., osd pool default size). So for
	# 10 OSDs and osd pool default size = 4, we'd recommend approximately
	# (100 * 10) / 4 = 250.

	osd pool default pg num = 250
	osd pool default pgp num = 250
```


## 参考

- [https://cloud.tencent.com/developer/article/1006084](https://cloud.tencent.com/developer/article/1006084)
- [https://cloud.tencent.com/developer/article/1006315](https://cloud.tencent.com/developer/article/1006315)
- [http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mon/](http://docs.ceph.com/docs/mimic/rados/deployment/ceph-deploy-mon/)
- [https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/pdf/administration_guide/Red_Hat_Ceph_Storage-2-Administration_Guide-en-US.pdf](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/pdf/administration_guide/Red_Hat_Ceph_Storage-2-Administration_Guide-en-US.pdf)
