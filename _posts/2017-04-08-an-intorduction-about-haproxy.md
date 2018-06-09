---
title: HAProxy简介
category: linux
tags:
- linux
---

## 介绍

HAProxy是一款提供高可用性、负载均衡的代理软件, 本文将对基本原理和配置进行介绍。

## 基本结构

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/haproxy_1.png"/>
</div>


<!--more-->


## 特点

- 免费开源，高性能，同时稳定性也好
- 支持TCP（第四层）和HTTP（第七层）应用
- 支持连接拒绝
- 支持全透明代理,可以用客户端IP地址或者任何其他地址来连接后端服务器
- 可用于email/mysql等非web的负载均衡
- 自带强大的监控服务器状态的页面
- 支持虚拟主机


### 配置

haproxy 配置中分成五部分内容，分别如下：

- global：参数是进程级的，通常是和操作系统相关
- defaults：配置默认参数，这些参数可以被用到frontend，backend，Listen组件
- frontend：接收请求的前端虚拟节点，frontend可以根据规则直接指定具体使用后端的backend
- backend：后端服务集群的配置，是真实服务器，一个backend对应一个或者多个实体服务器
- Listen: fronted和backend的组合体

```
global   # 全局参数的设置
    log 127.0.0.1 local0 info
    # log语法：log <address_1>[max_level_1] # 全局的日志配置，使用log关键字，指定使用127.0.0.1上的syslog服务中的local0日志设备，记录日志等级为info的日志

    user haproxy
    group haproxy
    # 设置运行haproxy的用户和组，也可使用uid，gid关键字替代

    daemon
    # 以守护进程的方式运行

    nbproc 16
    # 设置haproxy启动时的进程数，根据官方文档的解释，将其理解为：该值的设置应该和服务器的CPU核心数一致

    maxconn 4096
    # 定义每个haproxy进程的最大连接数 ，由于每个连接包括一个客户端和一个服务器端，所以单个进程的TCP会话最大数目将是该值的两倍。

    pidfile /var/run/haproxy.pid
    # 定义haproxy的pid

defaults # 默认部分的定义
    mode http
    # mode语法：mode {http|tcp|health} 。http是七层模式，tcp是四层模式，health是健康检测，返回OK

    log 127.0.0.1 local3 err
    # 使用127.0.0.1上的syslog服务的local3设备记录错误信息

    retries 3
    # 定义连接后端服务器的失败重连次数，连接失败次数超过此值后将会将对应后端服务器标记为不可用

    option httplog
    # 启用日志记录HTTP请求，默认haproxy日志记录是不记录HTTP请求的，只记录“时间[Jan 5 13:23:46] 日志服务器[127.0.0.1] 实例名已经pid[haproxy[25218]] 信息[Proxy http_80_in stopped.]”，日志格式很简单。

    option redispatch
    # 当使用了cookie时，haproxy将会将其请求的后端服务器的serverID插入到cookie中，以保证会话的SESSION持久性；而此时，如果后端的服务器宕掉了，但是客户端的cookie是不会刷新的，
    # 如果设置此参数，将会将客户的请求强制定向到另外一个后端server上，以保证服务的正常。

    option abortonclose
    # 当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接

    option dontlognull
    # 启用该项，日志中将不会记录空连接

    option httpclose
    # 每次请求完毕后主动关闭http通道

    timeout http-request    10s
    # http请求超时时间

    timeout queue           1m
    # 一个请求在队列里的超时时间

    timeout connect         10s
    # 设置成功连接到一台服务器的最长等待时间

    timeout client          1m
    # 设置连接客户端发送数据时的成功连接最长等待时间

    timeout server          1m
    # 设置服务器端回应客户度数据发送的最长等待时间

    timeout http-keep-alive 10s
    # 设置http-keep-alive的超时时间

    timeout check           10s
    # 检测超时


listen status # 定义一个名为status的部分
    bind 0.0.0.0:1080
    # 定义监听的套接字

    mode http
    # 定义为HTTP模式

    log global
    # 继承global中log的定义

    stats refresh 30s
    # stats是haproxy的一个统计页面的套接字，该参数设置统计页面的刷新间隔为30s

    stats uri /haproxy
    # 设置统计页面的uri为/haproxy 如果无法访问，检查selinux和iptables端口是否打开

    stats realm Private lands
    # 设置统计页面认证时的提示内容

    stats auth admin:rootroot
    # stats auth allen:rootroot
    # 设置统计页面认证的用户和密码为admin/rootroot，如果要设置多个，另起一行写入即可

    stats hide-version
    # 隐藏统计页面上的haproxy版本信息

frontend http_80_in # 定义一个名为http_80_in的前端部分
    bind 0.0.0.0:80
    # http_80_in定义前端部分监听的套接字

    mode http
    # 定义为HTTP模式

    log global
    # 继承global中log的定义

    option forwardfor
    # 启用X-Forwarded-For，在requests头部插入客户端IP发送给后端的server，使后端server获取到客户端的真实IP

    # acl static_down nbsrv(static_server) lt 1
    # 定义一个名叫static_down的acl，当backend static_sever中存活机器数小于1时会被匹配到

    acl php_web url_reg /*.php$
    # 定义一个名叫php_web的acl，当请求的url末尾是以.php结尾的，将会被匹配到

    acl static_web url_reg /*.(css|jpg|png|jpeg|js|gif)$
    # acl static_web path_end .gif .png .jpg .css .js .jpeg
    # 定义一个名叫static_web的acl，当请求的url末尾是以.css、.jpg、.png、.jpeg、.js、.gif结尾的，将会被匹配到，上面两种写法任选其一

    use_backend php_server if static_down
    # 如果满足策略static_down时，就将请求交予backend php_server

    use_backend php_server if php_web
    # 如果满足策略php_web时，就将请求交予backend php_server

    use_backend static_server if static_web
    # 如果满足策略static_web时，就将请求交予backend static_server

    default_backend static_server
    # 如果都不满足，就将请求交予backend static_server

backend php_server #定义一个名为php_server的后端部分
    mode http
    # 设置为http模式

    balance roundrobin
    # 设置haproxy的调度算法为轮询

    cookie SERVERID
    # 允许向cookie插入SERVER-ID，每台服务器的SERVER-ID可在下面使用cookie关键字定义

    option httpchk GET /test/index.php
    # 开启对后端服务器的健康检测，通过GET /test/index.php来判断后端服务器的健康情况

    server php_server_1 10.12.25.68:80 cookie 1 check inter 2000 rise 3 fall 3 weight 2
    server php_server_2 10.12.25.72:80 cookie 2 check inter 2000 rise 3 fall 3 weight 1
    server php_server_bak 10.12.25.79:80 cookie 3 check inter 1500 rise 3 fall 3 backup
    # server语法：server [:port] [param*]
    # 使用server关键字来设置后端服务器；为后端服务器所设置的内部名称[php_server_1]，该名称将会呈现在日志或警报中、后端服务器的IP地址，
    # 支持端口映射[10.12.25.68:80]、指定该服务器的SERVER-ID为1[cookie 1]、接受健康监测[check]、监测的间隔时长，单位毫秒[inter 2000]、监测正常多少次后被认为后端服务器是可用的[rise 3]、
    # 监测失败多少次后被认为后端服务器是不可用的[fall 3]、分发的权重[weight 2]、最后为备份用的后端服务器，当正常的服务器全部都宕机后，才会启用备份服务器[backup]

backend static_server
    mode http
    option httpchk GET /test/index.html
    server static_server_1 10.12.25.83:80 cookie 3 check inter 2000 rise 3 fall 3

```

## 实际配置

实际使用的测试配置文件:

```
# cat /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000


listen status # 定义一个名为status的部分
    bind 0.0.0.0:1090
    # 定义监听的套接字

    mode http
    # 定义为HTTP模式

    log global
    # 继承global中log的定义

    stats refresh 30s
    # stats是haproxy的一个统计页面的套接字，该参数设置统计页面的刷新间隔为30s

    stats uri /haproxy
    # 设置统计页面的uri为/admin?stats, 如果无法访问，检查selinux和iptables端口是否打开

    stats realm Private lands
    # 设置统计页面认证时的提示内容

    stats auth admin:password
    stats auth allen:rootroot
    # 设置统计页面认证的用户和密码，如果要设置多个，另起一行写入即可

    stats hide-version
    # 隐藏统计页面上的haproxy版本信息


#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:5000
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             app

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 192.168.60.39:88 check
    server  app2 192.168.60.44:88 check
    server  app3 192.168.60.43:88 check

#
```

可以通过URL: IP:1090/haproxy来访问haproxy的静态统计页面

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/haproxy_2.png"/>
</div>


## 参考

- [https://blog.csdn.net/tantexian/article/details/50056199](https://blog.csdn.net/tantexian/article/details/50056199)
- [https://blog.csdn.net/mini_xiang/article/details/60709470](https://blog.csdn.net/mini_xiang/article/details/60709470)
- [https://www.busyboy.cn/?p=508](https://www.busyboy.cn/?p=508)
- [https://www.digitalocean.com/community/tutorials/an-introduction-to-haproxy-and-load-balancing-concepts](https://www.digitalocean.com/community/tutorials/an-introduction-to-haproxy-and-load-balancing-concepts)
