---
title: keepalived简介
category: linux
tags:
- linux
---

## 介绍

本文将对keepalived的工作原理和集群配置进行介绍。

## 工作原理

- keepalived是集群管理中保证集群高可用的一个服务软件，其功能类似于heartbeat，用来防止单点故障。

- keepalived主要用作RealServer的健康状态检查以及LoadBalance主机和BackUP主机之间failover的实现。

- keepalived是以VRRP(Virtual Router Redundancy Protocol，即虚拟路由冗余协议)为基础来实现的，VRRP可以认为是实现路由器高可用的协议，
即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip(virtual ip)，
master会发组播，当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master,这样的话就可以保证路由器的高可用了。

- VRRP每个节点是有自己的优先级的，一般优先级是从0-255，数字越大优先级越高。

- keepalived支持LVS功能，与nginx/haproxy是七层负载均衡不通，LVS是四层负载均衡。


<!--more-->


### keepalived 架构

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/keepalived_1.png"/>
</div>


其中:
- VRRP组件：提供Director的故障转移功能从而实现Director的高可用。该组件可独立提供功能，无需LVS的支持。
- Checkers组件：负责RealServer的健康状况检查，并在LVS的拓扑中移除、添加RealServer。
- System Call组件：提供读取自定义脚本的功能。
- IPVS wrapper组件：负责将配置文件中IPVS相关规则发送到内核的ipvs模块。
- Netlink Reflector：用来设定、监控vrrp的vip地址。


### 核心组成部分

keepalived主要有三个模块，分别是core、check和vrrp:
- core模块: 为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析
- vrrp模块: 是来实现VRRP协议的
- check模块: 负责健康检查，包括常见的各种检查方式

keepalived在启动时会生成三个进程，一个主进程，两个子进程

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      1149  0.0  0.0 118608  1384 ?        Ss   Jun01   0:21 /usr/sbin/keepalived -D
root      1151  0.0  0.0 118608  2612 ?        S    Jun01   0:21  \_ /usr/sbin/keepalived -D
root      1152  0.0  0.0 120732  2456 ?        S    Jun01   4:39  \_ /usr/sbin/keepalived -D
```

## keepalived配置

keepalived服务的主配置文件为keepalived.conf，其配置主要分为如下几部分:

* Globals configurations
    * Global definitions
    * Static addresses
    * Static routes
* VRRP configuration
    * VRRP scripts
    * VRRP synchronization group
    * VRRP instance
* LVS configuration
    * Virtual server group
    * Virtual server


- 全局配置就是对整个keepalived生效的配置，VRRP配置是keepalived的核心，如果仅使用keepalived来做HA，LVS不需要配置。
- 配置文件都是以块（block）形式组织的，每个块都在{}范围内，#和!表示注释。


### 全局配置

#### global_defs
```
global_defs {
    notification_email {
       root@localhost                          # 邮件接收人员信息，可以为多个
       ...
    }
    notification_email_from alert@localhost    # 邮件发送者地址,可以伪装
    smtp_server  127.0.0.1                     # 通知邮件的smtp地址
    smtp_connect_timeout 30                    # 邮件发送连接超时时间，单位秒
    router_id myhostname                       # 机器标识, 通常为hostname
}
```

#### static_ipaddress/static_routes

static_ipaddress和static_routes区域配置的是本节点的IP和路由信息。正常情况下，机器都会有IP地址和路由信息的，因此不需要配置。


### VRRP配置

#### vrrp_script

用来做健康检查的

```
vrrp_script check_alived {
  script "/path/to/script.sh"    # 指定执行健康检查的脚本
  interval 2                     # 脚本执行时间间隔
  weight -2                      # 检查失败时，会将vrrp_instance的priority减少相应的值
  fall 2                         # 指定连续检测多少次失败才算真的失败
  rise 2                         # 指定连续检测多少次成功才算真的成功
}
```

#### vrrp_sync_group

vrrp_rsync_group用来定义vrrp_intance组，使得这个组内成员动作一致,当两个vrrp_instance同属于一个vrrp_rsync_group，
那么其中一个vrrp_instance发生故障切换时，另一个vrrp_instance也会跟着切换（即使这个instance没有发生故障）

```
vrrp_sync_group <STRING> {          # VRRP sync group declaration
    group {                         # group of instance to sync together
      project1                      # 这里定义的project1/project2是实例名，与下面的VRRP实例配置对应
      project2
      ...
    }
    notify_master /path/to/to_master.sh      # 切换到master状态时，要执行的脚本
    notify_backup /path_to/to_backup.sh      # 切换到backup状态时，要执行的脚本
    notify_fault /path/to/fault.sh           # 出错时，要执行的脚本
    notify /path/to/notify.sh                # 任意状态切换，都执行后面的脚本
    smtp_alert                               # 有切换时，进行全局配置中的邮件发送通知
}
```


#### vrrp_instance

vrrp_instance用来定义对外提供服务的VIP区域及其相关属性

```

vrrp_instance VI_1 {
    state MASTER           # 定义实例的角色状态是MASTER还是BACKUP
    interface eth0         # 定义vrrp绑定的接口，即接收或发送心跳通告的接口
    virtual_router_id 51   # 虚拟路由标识(VRID)，同一实例该数值必须相同，即master和backup中该值相同,取值范围0-255
    priority 100           # 该vrrp实例中本机的keepalived的优先级，优先级最高的为master
    advert_int 1           # 心跳信息发送和接收时间间隔，单位为秒
    authentication {       # 认证方式，同一实例中这个配置必须完全一样才可通过认证。建议使用PASS认证
        auth_type PASS
        auth_pass Asdf     # 最多支持8字符，超过8字符将只取前8字符
    }
    virtual_ipaddress {    # 设置的VIP, 只有master节点才会设置。master出现故障后，VIP会故障转移到backup
        192.168.200.100/24 dev eth0
    }
    track_script {         # 健康检查，与vrrp_script名称保持一致
        check_alived
    }

    notify_master /path/to_master.sh
    notify_backup /path/to_master.sh
    notify_fault /path/to_master.sh
    notify /path/to_master.sh
    smtp_alert
}
```

注意：
- 同一网段中virtual_router_id的值不能重复，否则会出错
- 可以用这条命令来查看该网络中所存在的vrid：`tcpdump -nn -i any net 224.0.0.0/8`


### LVS配置

```
virtual_server 192.168.200.100 443 { # 虚拟服务地址和端口，使用空格分隔，其中地址为VIP
    delay_loop 6                     # 健康检查时间间隔,即服务轮询的时间间隔
    lb_algo rr                       # 定义负载均衡LB的算法，有rr|wrr|lc|wlc|lblc|sh|dh多种
    lb_kind NAT                      # lvs的模型，有NAT/DR/TUN三种
    nat_mask 255.255.255.0
    persistence_timeout 50           # 持久会话保持时长
    protocol TCP                     # 监控服务的协议类型

    sorry_server 192.168.200.99 80   # 备用机，当所有后端realserver节点都不可用时，临时把所有的请求都发送到这里

    real_server 192.168.201.100 443 {   # 定义real_server部分，地址和端口使用空格分隔
        weight 1                        # LVS权重
        SSL_GET {                       # 健康状况检查的检查方式，常见的有HTTP_GET|SSL_GET|TCP_CHECK|MISC_CHECK。
            url {
              path /                    # 指定ssl_get健康状况检查的路径，例如检查index.html是否正常
              status_code 200           # 健康状况需要状态码，可以是status_code、digest或digest+status_code
                                        # digest值用keepalived的genhash命令生成，一般使用status_code即可
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3           # 表示3秒无响应就超时，即此realserver不健康，需重试连接
            nb_get_retry 3              # 表示重试3次，3次之后都超时就是宕机，防止误伤(nb=number)
            delay_before_retry 3        # 重试的时间间隔
                                        # 上述配置需12秒才能判断节点故障，时间太久，应改小
        }

        notify_up /path/to/up.sh        # 检查服务器正常(UP)后，要执行的脚本
        notify_down /path/to/down.sh    # 检查服务器失败(down)后，要执行的脚本
    }

    real_server 192.168.202.100 443 {
        weight 1
        HTTP_GET {
             url {
               path /
               state_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```


## 实际配置

以为下真实测试环境下的完整配置：

|role|ip|
|---|---|
|master|192.168.50.10|
|worker|192.168.50.11|
|worker|192.168.50.12|


- 192.168.50.10上的配置

```
$ cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL
}

vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 2
  weight -2
  fall 2
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 102
    authentication {
        auth_type PASS
        auth_pass Asdf
    }
    virtual_ipaddress {
        192.168.50.100/24 dev eth0
    }
    track_script {
        check_apiserver
    }
}
```

- 192.168.50.11上的配置

```
$ cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL
}

vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 2
  weight -2
  fall 2
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 100
    authentication {
        auth_type PASS
        auth_pass Asdf
    }
    virtual_ipaddress {
        192.168.50.100/24 dev eth0
    }
    track_script {
        check_apiserver
    }
}
```

- 192.168.50.12上的配置

```
$ cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL
}

vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 2
  weight -2
  fall 2
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 98
    authentication {
        auth_type PASS
        auth_pass Asdf
    }
    virtual_ipaddress {
        192.168.50.100/24 dev eth0
    }
    track_script {
        check_apiserver
    }
}
```


## 参考
- [http://www.keepalived.org/doc/index.html](http://www.keepalived.org/doc/index.html)
- [http://www.361way.com/keepalived-framework/5208.html](http://www.361way.com/keepalived-framework/5208.html)
- [http://outofmemory.cn/wiki/keepalived-configuration](http://outofmemory.cn/wiki/keepalived-configuration)
