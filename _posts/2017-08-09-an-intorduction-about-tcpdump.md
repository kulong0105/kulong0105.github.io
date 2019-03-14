---
title: tcpdump使用简介
category: linux
tags:
- linux
---

## 介绍

tcpdump是一款强大的网络抓包工具，运行在 linux 平台上。熟悉tcpdump的使用能够帮助分析、调试网络问题,
本文将对网络数据包格式和tcpdump工具的使用进行介绍。

<!--more-->


## 数据包格式

### TCP

TCP是一种可靠的、面向连接的字节流服务。源主机在传送数据前需要先和目标主机建立连接, 然后在此连接上，被编号的数据段按序收发。
同时，要求对每个数据段进行确认，从而保证了可靠性。TCP头部结构如下：

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/tcpdump_tcp.png"/>
</div>

TCP段首部的定长部分为20个字节:
- 16 位源端口与目标端口号: 用于标识发送端应用程序和接收端应用程序
- 32 位序号: 用来标识从TC 发送端向TC 接收端发送的数据字节流，表示在这个报文段中的的第一个数据字节
- 32 位确认序号: 用于表示期望收到的下一个序号，ACK=1 时有效。
- 4 位置首部长度: 计量单位为32bit
- 6 位保留字段
- 6 位控制字段：
    - URG: 紧急标志，和紧急指针配合使用，当其为1时表示，此报文要尽快传送
    - ACK: 确认标志，和确认号字段配合使用，当ACK位置1时，确认号字段有效
    - PSH: 推送标志，置1时，发送方将立即发送缓冲区中的数据
    - RST: 复位标志，置1时，表明有严重差错，必须释放连接重新建立
    - SYN: 同步标志，置1时，表示请求建立连接
    - FIN: 终止标志，置1时，表明数据已经发送完，请求释放连接
- 16 位窗口字段: 与TCP 的滑动窗口流量控制有关
- 16 位校验和: 覆盖了整个的TCP 报文段，包括首部和数据
- 16 位紧急指针: 和序号字段中的值相加表示紧急数据最后一个字节的序号
- 选项字段: 最常见的可选字段是MSS(Maximum Segment Size)最长报文大小, 指明本端所能接收的最大长度的报文段


### UDP

UDP是一种不可靠的、无连接的数据报服务。源主机在传送数据前不需要和目标主机建立连接。数据被冠以源、目标端口号等UDP报头字段后直接发往目的主机，
每个数据段的可靠性依靠上层协议来保证。在传送数据较小的情况下，UDP比TCP更加高效。UDP头部结构如下：

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/tcpdump_udp.png"/>
</div>

说明：
- 源/目标端口号字段：占16位，作用与TCP数据段中的端口号字段相同，用来标识源端和目标端的应用进程
- 长度字段：占16位，标明UDP头部和UDP数据的总长度字节
- 校验和字段：占16位，用来对UDP头部和UDP数据进行校验


### IP

IP协议处于TCP/IP四层模型的第二层（网络层）, 与ICMP/ARP/RARP处于同一层(但其数据是封装在IP数据包中的)，IP头部结构结构如下：

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/tcpdump_ip.png"/>
</div>

说明：
- 4位版本号: 指协议版本号，值为4代表 IPv4，下一代IP 协议的版本号为6
- 4位首部长度: 指的是包括选项字段在内的 IP 首部长度，由于是 4bit，所以 IP 首部最长只能是 60 字节（15 * 4）。
- 8位服务类型: 包括一个3 位的优先权字段（COS，Class of Service），4 位TOS 字段和1 位未用位。4 位TOS 分别代表:
    - 最小时延(D)
    - 最大吞吐量(T)
    - 最高可靠性(R)
    - 最小费用(C)
- 16位IP数据包长度: 包括首部和数据部分, 能表示的最大长度为65535，当IP数据包小于46字节时在以太网帧中数据将会被填充到46字节
- 16位标识字段: 是数据包的唯一标识，通常主机每发送一个数据包就会+1 ，在分片时会被复制到每一个分片中
- 3位标志字段和13位(片)偏移字段: 用于数据包分片和重组, 13 位(片)偏移字段，指示了这个分片在所属数据包中的位置, 3 位标志字段:
    - 0bit 保留
    - 1bit 为 DF: 0表示可以分片，1表示不能分片
    - 2bit 为 MF: 0表示最后一个分片，1表示还有分片。
- 8位生存时间(TTL): 设置了数据包可以经过的最多路由器数量
- 8位协议字段: 确定在数据包内传送的上层协议，1表示为ICMP，2表示为IGMP，6表示为TCP, 17表示为UDP
- 16位首部校验和: 根据IP首部计算的检验和码，它不对首部后面的数据进行计算
- 32位源/目标IP地址: 标识数据包的源端设备和目的端设备
- IP选项字段: 以32bit作为计量单位，不满32bit需要填充0, 最大值为40字节


### 以太网帧

在网络上传输的数据格式是以太网帧，其格式如下：

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/tcpdump_frame_1.png"/>
</div>


### 数据封装

应用程序使用TCP/IP协议传输应用数据时候，数据要被送入协议栈经过逐层封装，最终作为比特流在媒体上传送出去, 其过程如下所示：

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/tcpdump_frame_2.png"/>
</div>

- 以太网帧中的数据长度规定最小46 字节，最大1500 字节，ARP 和RARP 数据包的长度不够46 字节，要在后面补填充位
- 最大值1500 称为以太网的最大传输单元（MTU), 当IP数据包长度大于 MTU 时会被拆成多个帧传输，称为"IP分片"


## tcpdump使用

### 常用选项

| 序号 | 选项 | 说明 |
| ---  | ---  | ---  |
| 1 | -i | 指定监听的网络接口|
| 2 | -n | 不解析域名，直接显示ip |
| 3 | -nn | 不解析协议和端口号到名称，直接使用数字|
| 4 | -c number | 截取 number 个报文，然后结束|
| 5 | -w filename | 写入文件 |
| 6 | -r filename | 读取文件 |
| 7 | -t | 不显示时间戳 |
| 8 | -s | 设置捕捉每个数据包的长度 |
| 9 | -A | 使用ascii显示报文的内容|
| 10 | -X | 同时用 hex 和 ascii 显 示报文的内容 |
| 11 | -S | 显示绝对的序列号，而不是相对编号 |
| 12 | -e | 打印出数据链路层的头部信息，包括源mac和目的mac |
| 13| -l | 使标准输出变为缓冲行形式|
| 14| -v | 显示详细信息|
| 15| -D | 显示可监听的网络接口|


### 过滤器

网络数据包异常的多，绝大多时候，只需要获取所关心的数据包，可使用过滤器获取指定的数据包, 过滤器分为多种:

- 类型: `host`, `net`, `port`, `portrange`
- 方向：`src`, `dst`
- 协议：`tcp`, `udp`, `icmp`, `ssh`
- 表达式: `and` 或者 `&&`, `or` 或者 `||`, `not` 或者 `!`
- 数据包大小：`greater`, `less`


### 理解输出信息

```
[renyl@localhost ~]$ sudo tcpdump -nnnSv
tcpdump: listening on wlp3s0, link-type EN10MB (Ethernet), capture size 262144 bytes
10:13:07.137518 IP (tos 0x0, ttl 63, id 37929, offset 0, flags [DF], proto TCP (6), length 89)
    192.168.20.30.8065 > 192.168.31.60.42426: Flags [P.], cksum 0x601c (correct), seq 216738098:216738135, ack 2219416544, win 403, options [nop,nop,TS val 408252304 ecr 1132601724], length 37
10:13:07.177754 IP (tos 0x0, ttl 64, id 14462, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.31.60.42426 > 192.168.20.30.8065: Flags [.], cksum 0x8e52 (correct), ack 216738135, win 647, options [nop,nop,TS val 1132612137 ecr 408252304], length 0
10:13:07.819779 IP (tos 0x0, ttl 64, id 14463, offset 0, flags [DF], proto TCP (6), length 95)
```

说明：
- 在时间点10:13:07.137518发送IP数据包
- 源IP地址192.168.20.30，端口号8065，向目标IP地址192.168.31.60，端口号42426发送数据包
- Flag的含义：
    - `[S]`： SYN（开始连接）
    - `[S.]`: SYN-ACK
    - `[.]`: No Flag Set
    - `[P]`: PSH（推送数据）
    - `[F]`: FIN （结束连接）
    - `[R]`: RST（重置连接）


### 实例

注： 需要具有root权限才可以运行tcpdump命令

- 监视所有网络接口上流过的数据包

```
tcpdump -i any
```
注：默认只会选择机器上的`一块网卡`进行监听

- 监视指定网络接口的数据包

```
tcpdump -i eth1
```

- 显示完整的时间戳

```
tcpdump -ttttnnv
```

- 截获主机192.168.60.10和主机192.168.60.11或192.168.60.12的通信

```
tcpdump host 192.168.60.10 and \(192.168.60.11 or 192.168.60.12\)
```

- 打印allen与任何其他主机之间通信的IP数据包, 但不包括与lucky之间的数据包

```
tcpdump ip host allen and not lucky
```

- 抓取包含192.168.60.10的数据包

```
# tcpdump -i eth0 -vnn host 192.168.60.10
```

- 抓取包含192.168.60.0/24网段的数据包

```
# tcpdump -i eth0 -vnn net 192.168.60.0/24
```

- 抓取包含端口22的数据包

```
# tcpdump -i eth0 -vnn port 22
```

- 抓取包含端口22-125的数据包

```
# tcpdump -i eth0 -vnn portrange 22-125
```

- 抓取udp协议的数据包

```
# tcpdump -i eth0 -vnn  udp
```

- 抓取icmp协议的数据包

```
# tcpdump -i eth0 -vnn icmp
```

- 抓取ssh协议的数据包

```
# tcpdump ssh
```

- 抓取arp协议的数据包

```
# tcpdump -i eth0 -vnn arp
```

- 抓取ip协议的数据包

```
# tcpdump -i eth0 -vnn ip
```

- 抓取源ip是192.168.60.12数据包。

```
# tcpdump -i eth0 -vnn src host 192.168.60.12
```

- 抓取目的ip是192.168.60.12数据包

```
# tcpdump -i eth0 -vnn dst host 192.168.60.12
```

- 抓取源ip是192.168.60.13且目的ip是22的数据包

```
# tcpdump -i eth0 -vnn src host 192.168.60.13 and dst port 22
```

- 抓取源ip是192.168.60.12且端口不是22的数据包

```
# tcpdump -i eth0 -vnn src host 192.168.60.12 and not port 22
```

- 抓取源ip是192.168.60.11且目的端口是22，或源ip是192.168.60.12且目的端口是80的数据包

```
# tcpdump -i eth0 -vnn 'src host 192.168.60.11 and dst port 22 ' or 'src host 192.168.60.12 and dst port 80'
```

- 把抓取的数据包记录存到/tmp/tcpdump文件中，当抓取100个数据包后就退出程序

```
# tcpdump –i eth0 -vnn -w /tmp/tcpdump -c 100
```

- 从/tmp/tcpdump记录中读取tcp协议的数据包

```
# tcpdump –i eth0 -vnn -r /tmp/tcpdump tcp
```

- 从/tmp/tcpdump记录中读取包含192.168.60.12的数据包

```
# tcpdump –i eth0 -vnn -r  /tmp/tcpdump host 192.168.60.12
```

- 获取数据包大小大于128字节的

```
# tcpdump -nnvS greater 128
```

- 获取数据包大小小于128字节的

```
# tcpdump -nnvS less 128
```

- 获取所有URGENT(URG)数据包

```
# tcpdump 'tcp[13] & 32!=0'
```

- 获取所有ACKNOWLEDGE(ACK)数据包

```
# tcpdump 'tcp[13] & 16!=0'
```

- 获取HTTP User Agent来自HTTP请求包

```
tcpdump -nn -A -s1500 -l | grep "User-Agent:"
```

- 获取密码来自HTTP POST数据

```
tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"
```


## 参考
- [https://strawhatfy.github.io/2015/07/30/TCP-IP-Protocol/](https://strawhatfy.github.io/2015/07/30/TCP-IP-Protocol/)
- [https://blog.csdn.net/laoniu_c/article/details/39269165](https://blog.csdn.net/laoniu_c/article/details/39269165)
- [https://hackertarget.com/tcpdump-examples/](https://hackertarget.com/tcpdump-examples/)
- [http://man.linuxde.net/tcpdump](http://man.linuxde.net/tcpdump)
- [http://bencane.com/2014/10/13/quick-and-practical-reference-for-tcpdump/](http://bencane.com/2014/10/13/quick-and-practical-reference-for-tcpdump/)
- [https://danielmiessler.com/study/tcpdump/](https://danielmiessler.com/study/tcpdump/)
- [https://linuxwiki.github.io/NetTools/tcpdump.html](https://linuxwiki.github.io/NetTools/tcpdump.html)
