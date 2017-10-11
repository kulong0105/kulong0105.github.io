---
title: iptables原理和实践
category: linux
tags:
- linux
---

## 介绍

本文将对iptables的基本原理进行介绍，并进行一些实践操作。

## 基本原理

### netfilter模块

iptables被用来配置系统的防火墙，底层是通过Linux操作系统的内核模块netfilter来完成的,
netfilter可实现安全策略中的许多功能：
* 数据包过滤
* 数据包处理
* 地址伪装
* 透明代理
* 动态网络地址转换(NAT)
* 包速率限制
* 负载均衡

iptables可以灵活的组织这些功能，从而形成强大的防火墙。

<!--more-->

netfilter模块中实现了数据包的5个挂载点（HOOK POINT，数据据包到达这些位置的时候会主动调用相关函数，
使的能在数据包路由的时候改变它们的方向/内容):
* PRE_ROUTING
* INPUT
* OUTPUT
* FORWARD
* POST_ROUTING


netfilter所设置的规则是放在内核内存中的, iptables是一个应用层程序，通过netfilter提供的接口对存放在内存
中的netfilter配置表进行修改，这个配置表由：表tables、链chains、规则rules 组成，iptables在应用层负责修改
这个规则文件。(类似的应用程序还有firewalld)

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/iptables_1.jpg"/>
</div>


### iptables的组成

iptables可以看作由`四表五链` 组成:

四表：
* filter表：主要用于对数据包进行过滤，根据具体的规则决定是否放行该数据包（如DROP、ACCEPT、REJECT、LOG）

* nat表: 主要用于修改数据包的IP地址、端口号等信息（网络地址转换，如SNAT、DNAT、MASQUERADE、REDIRECT）。
		 属于一个流的包(因为包的大小限制导致数据可能会被分成多个数据包)只会经过这个表一次。

* mangle表: 主要用于修改数据包的TOS（Type Of Service，服务类型）、TTL（Time To Live，生存周期）以及为数据
            包设置Mark标记,以实现Qos(Quality Of Service，服务质量)调整以及策略路由等应用。

* raw表：主要用于决定数据包是否被状态跟踪机制处理。在匹配数据包时，raw表的规则要优先于其他表。
	* iptables中数据包被跟踪连接的4种不同状态:
		* NEW: 该包想要开始一个连接

		* RELATED: 该包是属于某个已经建立的连接所建立的新连接。例如：FTP的数据传输连接就是控制连接所RELATED出来的连接,
           以及icmp-type 0 (ping应答) 就是icmp-type 8 (ping请求)所RELATED出来的。

		* ESTABLISHED: 只要发送并接到应答，一个数据连接从NEW变为ESTABLISHED,而且该状态会继续匹配这个连接的后续数据包。

		* INVALID: 数据包不能被识别属于哪个连接或没有任何状态比

NOTE: 表的处理优先级：raw>mangle>nat>filter, 默认表是filter（没有指定表的时候就是filter表）

五链：

* PREROUTING链：在对数据包作路由选择之前，应用此链中的规则，如DNAT。
* INPUT链：当接收到防火墙本机地址的数据包（入站）时，应用此链中的规则。
* OUTPUT链：当防火墙本机向外发送数据包（出站）时，应用此链中的规则。
* FORWARD链：当接收到需要通过防火墙发送给其他地址的数据包（转发）时，应用此链中的规则。
* POSTROUTING链：在对数据包作路由选择之后，应用此链中的规则，如SNAT。

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/iptables_2.jpg"/>
</div>

其中，INPUT、OUTPUT链更多的应用在“主机防火墙”中，即主要针对服务器本机进出数据的安全控制；
FORWARD、PREROUTING、POSTROUTING链更多的应用在“网络防火墙”中，特别是防火墙服务器作为网关使用。


### iptables数据包的处理方式

iptables针对数据包有如下几种处理方式:

* ACCEPT：允许数据包通过

* DROP：直接丢弃数据包，不给任何回应信息

* REJECT：拒绝数据包通过，必要时会给数据发送端一个响应的信息。

* SNAT：源地址转换。在进入路由层面的route之前，重新改写源地址，目标地址不变，并在本机建立NAT表项,当数据返回时，
        根据NAT表将目的地址数据改写为数据发送出去时候的源地址，并发送给主机。解决内网用户用同一个公网地址上网的问题。
	* MASQUERADE: 是SNAT的一种特殊形式，适用于像adsl这种临时会变的ip上

* DNAT:目标地址转换。和SNAT相反，IP包经过route之后、出本地的网络栈之前，重新修改目标地址，源地址不变，
       在本机建立NAT表项，当数据返回时，根据NAT表将源地址修改为数据发送过来时的目标地址，并发给远程主机。
	   可以隐藏后端服务器的真实地址。
	* REDIRECT：是DNAT的一种特殊形式，将网络包转发到本地host上（不管IP头部指定的目标地址是啥），方便在本机做端口转发。

* LOG：在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则

除去最后一个LOG，前3条规则匹配数据包后，该数据包不会再往下继续匹配了，所以编写的`规则顺序`极其关键。


### iptables的数据包路由原理

网口数据包由底层的网卡接收，通过数据链路层的解包之后(去除数据链路帧头)，就进入了TCP/IP协议栈和Netfilter
混合的数据包处理流程中了。数据包的接收、处理、转发流程构成一个有限状态向量机，经过一些列的内核处理函数、
以及Netfilter Hook点，最后被转发、或者本次上层的应用程序消化掉。

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/iptables_3.jpg"/>
</div>

* 当一个数据包进入网卡时，数据包首先进入PREROUTING链，在PREROUTING链中我们有机会修改数据包的目的IP，
  然后内核的”路由模块”根据”数据包目的IP”以及”内核中的路由表”判断是否需要转送出去

* 如果数据包就是进入本机的(即数据包的目的IP是本机的网口IP)，数据包就会沿着图向下移动，到达INPUT链。
数据包到达INPUT链后，任何进程都会收到它

* 本机上运行的程序也可以发送数据包，这些数据包经过OUTPUT链，然后到达POSTROTING链输出

* 如果数据包是要转发出去的(即目的IP地址不再当前子网中)，且内核允许转发，数据包就会向右移动，
  经过FORWARD链，然后到达POSTROUTING链输出(选择对应子网的网口发送出去)


## iptables编写规则

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/iptables_4.jpg"/>
</div>

1) [-t 表名]：该规则所操作的哪个表，可以使用filter、nat等，如果没有指定则默认为filter  
2) -A：新增一条规则，到该规则链列表的最后一行  
3) -I：插入一条规则，原本该位置上的规则会往后顺序移动，没有指定编号则为1  
4) -D：从规则链中删除一条规则，要么输入完整的规则，或者指定规则编号加以删除  
5) -R：替换某条规则，规则替换不会改变顺序，而且必须指定编号。  
6) -P：设置某条规则链的默认动作  
7) -nL：-L、-n，查看当前运行的防火墙规则列表  
8) chain名：指定规则表的哪个链，如INPUT、OUPUT、FORWARD、PREROUTING等  
9) [规则编号]：插入、删除、替换规则时用，—line-numbers显示号码  
10)[-i|o 网卡名称]：i是指定数据包从哪块网卡进入，o是指定数据包从哪块网卡输出  
11)[-p 协议类型]：可以指定规则应用的协议，包含tcp、udp和icmp等  
12)[-s 源IP地址]：源主机的IP地址或子网地址  
13)[--sport 源端口号]：数据包的IP的源端口号  
14)[-d目标IP地址]：目标主机的IP地址或子网地址  
15)[--dport目标端口号]：数据包的IP的目标端口号  
16)-m：extend matches，这个选项用于提供更多的匹配参数，如：  
17)-m state —state ESTABLISHED,RELATED  
18)-m tcp —dport 22  
19)-m multiport —dports 80,8080  
20)-m icmp —icmp-type 8  
21)<-j 动作>：处理数据包的动作，包括ACCEPT、DROP、REJECT等   


## iptables用法

* 查看现有规则
```bash
# iptables -L -n -v --line-numbers
```

* 删除所有现有规则
```bash
# iptables -F
```

* 设置默认的 chain 策略
```bash
# iptables -P INPUT DROP
# iptables -P FORWARD DROP
# iptables -P OUTPUT DROP
```

* 阻止某个特定的 IP 地址
```bash
# iptables -A INPUT -s $ip_addr -j DROP
```

* 允许全部入站的SSH
```bash
# iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

* 只允许某个特定网络进来的 SSH
```bash
# iptables -A INPUT -i eth0 -p tcp -s 192.168.200.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```


* 允许入站的HTTP
```bash
# iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

* 多端口（允许进来的 SSH、HTTP 和 HTTPS）
```bash
# iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT
```

* 允许出站的SSH
```bash
# iptables -A OUTPUT -o eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

* 允许出站的SSH，但仅访问某个特定的网络
```bash
# iptables -A OUTPUT -o eth0 -p tcp -d 192.168.101.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

* 允许出站的 HTTPS
```bash
# iptables -A OUTPUT -o eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A INPUT -i eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

* 对进来的 HTTPS 流量做负载均衡
```bash
# iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.101:443
# iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 1 -j DNAT --to-destination 192.168.1.102:443
# iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 2 -j DNAT --to-destination 192.168.1.103:443
```

* 允许从内部向外部PING
```bash
# iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
# iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

* 允许从外部向内部PING
```bash
# iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
# iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

* 允许转发除ICMP协议以外的所有数据包
```bash
# iptables -A FORWARD -p ! icmp -j ACCEPT
```
NOTE: 使用“！”可以将条件取反


* 允许环回（loopback）访问
```bash
# iptables -A INPUT -i lo -j ACCEPT
# iptables -A OUTPUT -o lo -j ACCEPT
```

* 允许 packets 从内网访问外网
```bash
# iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```
NOTE:
1) eth1 is connected to external network (internet), and eth0 is connected to internal network (192.168.1.x)
2) enable forward function: echo 1 > /proc/sys/net/ipv4/ip_forward


* 允许外出的DNS
```bash
# iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPT
# iptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```

* 允许某个特定网络 rsync 进入本机
```bash
# iptables -A INPUT -i eth0 -p tcp -s 192.168.101.0/24 --dport 873 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 873 -m state --state ESTABLISHED -j ACCEPT
```

* 允许 Sendmail 或 Postfix
```bash
# iptables -A INPUT -i eth0 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```

* 允许 IMAP 和 IMAPS
```bash
# iptables -A INPUT -i eth0 -p tcp --dport 143 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 143 -m state --state ESTABLISHED -j ACCEPT
# iptables -A INPUT -i eth0 -p tcp --dport 993 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 993 -m state --state ESTABLISHED -j ACCEPT
```

* 允许 POP3 和 POP3S
```bash
# iptables -A INPUT -i eth0 -p tcp --dport 110 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 110 -m state --state ESTABLISHED -j ACCEPT
# iptables -A INPUT -i eth0 -p tcp --dport 995 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 995 -m state --state ESTABLISHED -j ACCEPT
```

* 防止 DoS 攻击
```bash
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```
NOTE: 将连接限制到每分钟 25 个，上限设定为100


* 设置 422 端口转发到 22 端口
```bash
# iptables -t nat -A PREROUTING -p tcp -d 192.168.102.37 --dport 422 -j DNAT --to-destination 192.168.102.37:22
# iptables -A INPUT -i eth0 -p tcp --dport 422 -m state --state NEW,ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -o eth0 -p tcp --sport 422 -m state --state ESTABLISHED -j ACCEPT
```
或者
```bash
# iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 25 -j REDIRECT --to-port 2525		
```

* 屏蔽指定MAC地址
```bash
# iptables -A INPUT -m mac --mac-source $mac_addr -j DROP		
```

* 更换源IP地址
```bash
# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j SNAT --to-source 192.168.5.3
# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j SNAT --to-source 192.168.5.3-192.168.5.5
# iptables -t nat -A POSTROUTING -s 10.8.0.0/255.255.255.0 -o eth0 -j MASQUERADE
```
NOTE： 
1) --to-source 可以指定多个IP地址
2) MASQUERADE会自动读取eth0现在的ip地址然后做snat出去


* 限制并发连接数
```bash
# iptables -A INPUT -p tcp --syn --dport 22 -m connlimit --connlimit-above 3 -j REJECT		
```
NOTE: 限制每个客户端不超过 3 个连接。


* 保存iptables规则
```bash
# iptables-save
```
NOTE: 情况下，iptables 规则的操作会立即生效。但由于规则都是保存在内存当中的，
所以重启系统会造成配置丢失，要永久保存 IPtables 规则可以使用 iptables-save 命令。


* 为丢弃的包做日志（Log）
```bash
# iptables -N LOGGING //新建LOGGING chain
# iptables -A INPUT -j LOGGING
# iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables Packet Dropped: " --log-level 7
# iptables -A LOGGING -j DROP
```


## 参考
[iptables防火墙原理知多少](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655438389&idx=2&sn=951f77ec3c82e2d351f5a99c689cb467&chksm=bd730a428a048354050ce5a44bae93737f3ba868d0562f23440832e30ce59bdeac33714396df&scene=0&pass_ticket=hdYlVJ3H8OwZDRQZuKheVyBgST8d7tWgzLu4SUBecHLa%2FpHiqM75p1UX6f8W3QDT#rd)

[25 个常用的 Linux iptables 规则](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655438389&idx=2&sn=951f77ec3c82e2d351f5a99c689cb467&chksm=bd730a428a048354050ce5a44bae93737f3ba868d0562f23440832e30ce59bdeac33714396df&scene=0&pass_ticket=hdYlVJ3H8OwZDRQZuKheVyBgST8d7tWgzLu4SUBecHLa%2FpHiqM75p1UX6f8W3QDT#rd)
