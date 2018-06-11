---
title: etcd集群部署和使用简介
category: linux
tags:
- linux
---

## 介绍

etcd是由CoreOS开发的开源项目，其是一个高可用的Key/Value存储系统，主要用于共享配置和服务发现，本文将对etcd集群的配置和基本使用进行介绍

<!--more-->

## etcd集群部署

由于etcd集群使用raft协议保证高可用，所以etcd集群至少要配置3台机器，如下采用systemd方式进行部署

- 机器1（my_etcd_1)配置

```
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/usr/local/bin/etcd \
	--name my_etcd_1 \
	--data-dir /data/etcd \
	--listen-client-urls https://0.0.0.0:2379 \
	--advertise-client-urls https://192.168.60.21:2379 \
	--listen-peer-urls https://0.0.0.0:2380 \
	--initial-advertise-peer-urls https://192.168.60.21:2380 \
	--initial-cluster my_etcd_1=https://192.168.60.21:2380,my_etcd_2=https://192.168.60.22:2380,my_etcd_3=https://192.168.60.23:2380 \
	--initial-cluster-token 9477af68bbee1b9ae037d6fd9e7efefd \
	--initial-cluster-state new \
	--cert-file=/etc/pki/etcd/server.pem \
	--key-file=/etc/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/pki/etcd/ca.pem \
	--peer-cert-file=/etc/pki/etcd/peer.pem \
	--peer-key-file=/etc/pki/etcd/peer-key.pem \
	--peer-client-cert-auth \
	--peer-trusted-ca-file=/etc/pki/etcd/ca.pem

[Install]
WantedBy=multi-user.target
```

- 机器2（my_etcd_2)配置

```
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/usr/local/bin/etcd \
	--name my_etcd_2 \
	--data-dir /data/etcd \
	--listen-client-urls https://0.0.0.0:2379 \
	--advertise-client-urls https://192.168.60.22:2379 \
	--listen-peer-urls https://0.0.0.0:2380 \
	--initial-advertise-peer-urls https://192.168.60.22:2380 \
	--initial-cluster my_etcd_1=https://192.168.60.21:2380,my_etcd_2=https://192.168.60.22:2380,my_etcd_3=https://192.168.60.23:2380 \
	--initial-cluster-token 9477af68bbee1b9ae037d6fd9e7efefd \
	--initial-cluster-state new \
	--cert-file=/etc/pki/etcd/server.pem \
	--key-file=/etc/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/pki/etcd/ca.pem \
	--peer-cert-file=/etc/pki/etcd/peer.pem \
	--peer-key-file=/etc/pki/etcd/peer-key.pem \
	--peer-client-cert-auth \
	--peer-trusted-ca-file=/etc/pki/etcd/ca.pem

[Install]
WantedBy=multi-user.target
```

- 机器3（my_etcd_3)配置

```
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/usr/local/bin/etcd \
	--name my_etcd_3 \
	--data-dir /data/etcd \
	--listen-client-urls https://0.0.0.0:2379 \
	--advertise-client-urls https://192.168.60.23:2379 \
	--listen-peer-urls https://0.0.0.0:2380 \
	--initial-advertise-peer-urls https://192.168.60.23:2380 \
	--initial-cluster my_etcd_1=https://192.168.60.21:2380,my_etcd_2=https://192.168.60.22:2380,my_etcd_3=https://192.168.60.23:2380 \
	--initial-cluster-token 9477af68bbee1b9ae037d6fd9e7efefd \
	--initial-cluster-state new \
	--cert-file=/etc/pki/etcd/server.pem \
	--key-file=/etc/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/pki/etcd/ca.pem \
	--peer-cert-file=/etc/pki/etcd/peer.pem \
	--peer-key-file=/etc/pki/etcd/peer-key.pem \
	--peer-client-cert-auth \
	--peer-trusted-ca-file=/etc/pki/etcd/ca.pem

[Install]
WantedBy=multi-user.target
```

注：
- 启动单个机器上的etcd服务，将被hang住，这是正常现象，直到所有3台机器的etcd服务都启动后，systemd会根据其service文件中"Type=notify"配置，将其放到后台运行
- 本集群配置使用TLS加密，可使用CFSSL工具生成相关证书文件: [CFSSL使用](https://kulong0105.cn/linux/2017/06/10/an-intorduction-about-cert)或[官方指导](https://coreos.com/os/docs/latest/generate-self-signed-certificates.html)


## etcd使用

### 读/写key

- etcdctl

```
$ alias my_etcdctl="etcdctl --endpoints=https://192.168.60.21:2379 --ca-file=/etc/pki/etcd/ca.pem --cert-file=/etc/pki/etcd/client.pem --key-file=/etc/pki/etcd/client.pem"
$ my_etcdctl set /message hello
hello
$ my_etcdctl set /name kulong0105
hello
$ my_etcdctl ls
/message
/name
$ my_etcdctl get /message
hello
$ my_etcdctl rm /name
$ my_etcdctl rm /message
PrevNode.Value: hello
$ my_etcdctl get /message
Error:  100: Key not found (/message) [13]
$
```

- curl

```
$ alias my_curl="curl --cacert /etc/pki/etcd/ca.pem --cert /etc/pki/etcd/client.pem --key /etc/pki/etcd/client-key.pem"
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/message -d value="Hello"
{"action":"set","node":{"key":"/message","value":"Hello","modifiedIndex":14,"createdIndex":14}}
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/name -d value="kulong0105"
{"action":"set","node":{"key":"/name","value":"kulong0105","modifiedIndex":15,"createdIndex":15}}
$ my_curl https://192.168.60.21:2379/v2/keys/
{"action":"get","node":{"dir":true,"nodes":[{"key":"/message","value":"Hello","modifiedIndex":14,"createdIndex":14},{"key":"/name","value":"kulong0105","modifiedIndex":15,"createdIndex":15}]}}
$ my_curl https://192.168.60.21:2379/v2/keys/ | python -m json.tool
{
    "action": "get",
    "node": {
        "dir": true,
        "nodes": [
            {
                "createdIndex": 18,
                "key": "/message",
                "modifiedIndex": 18,
                "value": "hello"
            },
            {
                "createdIndex": 19,
                "key": "/name",
                "modifiedIndex": 19,
                "value": "kulong0105"
            }
        ]
    }
}
$ my_curl https://192.168.60.21:2379/v2/keys/name
{"action":"get","node":{"key":"/name","value":"kulong0105","modifiedIndex":15,"createdIndex":15}}
$ my_curl -X DELETE https://192.168.60.21:2379/v2/keys/name
{"action":"delete","node":{"key":"/name","modifiedIndex":16,"createdIndex":15},"prevNode":{"key":"/name","value":"kulong0105","modifiedIndex":15,"createdIndex":15}}
$ my_curl -X DELETE https://192.168.60.21:2379/v2/keys/
{"errorCode":107,"message":"Root is read only","cause":"/","index":16}
$ my_curl -X DELETE https://192.168.60.21:2379/v2/keys/message
{"action":"delete","node":{"key":"/message","modifiedIndex":17,"createdIndex":14},"prevNode":{"key":"/message","value":"Hello","modifiedIndex":14,"createdIndex":14}}
$ my_curl -X DELETE https://192.168.60.21:2379/v2/keys/message
{"errorCode":100,"message":"Key not found","cause":"/message","index":17}
$
```


### Test and Set

- etcdctl

```
$ alias my_etcdctl="etcdctl --endpoints=https://192.168.60.21:2379 --ca-file=/etc/pki/etcd/ca.pem --cert-file=/etc/pki/etcd/client.pem --key-file=/etc/pki/etcd/client.pem"
$ my_etcdctl  set /name "kulong0105"
kulong0105
$ my_etcdctl set /name "lucky" --swap-with-value "kulong0106"
Error:  101: Compare failed ([kulong0106 != kulong0105]) [37]
$ my_etcdctl set /name "lucky" --swap-with-value "kulong0105"
lucky
$ my_etcdctl get /name
lucky
$
```

- curl

```
$ alias my_curl="curl --cacert /etc/pki/etcd/ca.pem --cert /etc/pki/etcd/client.pem --key /etc/pki/etcd/client-key.pem"
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/name -d value="kulong0105"
{"action":"set","node":{"key":"/name","value":"kulong0105","modifiedIndex":40,"createdIndex":40}}
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/name?prevValue=kulong0106 -d value=lucky
{"errorCode":101,"message":"Compare failed","cause":"[kulong0106 != kulong0105]","index":40}
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/name?prevValue=kulong0105 -d value=lucky
{"action":"compareAndSwap","node":{"key":"/name","value":"lucky","modifiedIndex":41,"createdIndex":40},"prevNode":{"key":"/name","value":"kulong0105","modifiedIndex":40,"createdIndex":40}}
$ my_curl https://192.168.60.21:2379/v2/keys/name
{"action":"get","node":{"key":"/name","value":"lucky","modifiedIndex":41,"createdIndex":40}}
$
```


### 设置TTL

- etcdctl

```
$ alias my_etcdctl="etcdctl --endpoints=https://192.168.60.21:2379 --ca-file=/etc/pki/etcd/ca.pem --cert-file=/etc/pki/etcd/client.pem --key-file=/etc/pki/etcd/client.pem"
$ my_etcdctl set /time none --ttl 10 && sleep 11 && my_etcdctl get /time
10
Error:  100: Key not found (/time) [27]
$
```

- curl

```
$ alias my_curl="curl --cacert /etc/pki/etcd/ca.pem --cert /etc/pki/etcd/client.pem --key /etc/pki/etcd/client-key.pem"
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/time?ttl=10 -d value=none && sleep 11 && my_curl https://192.168.60.21:2379/v2/keys/time
{"action":"set","node":{"key":"/time","value":"none","expiration":"2018-06-11T11:30:42.497377652Z","ttl":10,"modifiedIndex":22,"createdIndex":22}}
{"errorCode":100,"message":"Key not found","cause":"/time","index":23}
$
```


### 创建目录

- etcd

```
$ alias my_etcdctl="etcdctl --endpoints=https://192.168.60.21:2379 --ca-file=/etc/pki/etcd/ca.pem --cert-file=/etc/pki/etcd/client.pem --key-file=/etc/pki/etcd/client.pem"
$ my_etcdctl mkdir /testdir
$ my_etcdctl set /testdir/name kulong0105
kulong0105
$ my_etcdctl ls /testdir
/testdir/name
$ my_etcdctl rmdir /testdir
Error:  108: Directory not empty (/testdir)
$ my_etcdctl rm /testdir/name
PrevNode.Value: kulong0105
$ my_etcdctl rmdir /testdir
$
```

- curl

```
$ alias my_curl="curl --cacert /etc/pki/etcd/ca.pem --cert /etc/pki/etcd/client.pem --key /etc/pki/etcd/client-key.pem"
$ my_curl -X PUT https://192.168.60.21:2379/v2/keys/testdir/name -d value="kulong0105"
{"action":"set","node":{"key":"/testdir/name","value":"kulong0105","modifiedIndex":30,"createdIndex":30}}
$ my_curl  https://192.168.60.21:2379/v2/keys/testdir
{"action":"get","node":{"key":"/testdir","dir":true,"nodes":[{"key":"/testdir/name","value":"kulong0105","modifiedIndex":30,"createdIndex":30}],"modifiedIndex":30,"createdIndex":30}}
$ my_curl  https://192.168.60.21:2379/v2/keys/testdir/name
{"action":"get","node":{"key":"/testdir/name","value":"kulong0105","modifiedIndex":30,"createdIndex":30}}
$ my_curl -X DELETE https://192.168.60.21:2379/v2/keys/testdir?recursive=true
{"action":"delete","node":{"key":"/testdir","dir":true,"modifiedIndex":47,"createdIndex":45},"prevNode":{"key":"/testdir","dir":true,"modifiedIndex":45,"createdIndex":45}}
$ my_curl  https://192.168.60.21:2379/v2/keys/testdir/name
{"errorCode":100,"message":"Key not found","cause":"/testdir/name","index":31}
$ my_curl  https://192.168.60.21:2379/v2/keys/testdir
{"errorCode":100,"message":"Key not found","cause":"/testdir","index":47}
$
$
```

注：
使用curl命令，可能出现出现错误
```
$ curl --cacert /etc/pki/etcd/ca.pem --cert /etc/pki/etcd/server.pem --key /etc/pki/etcd/server-key.pem  https://192.168.60.21:2379/v2/keys/name
curl: (58) unable to load client key: -8178 (SEC_ERROR_BAD_KEY)
$
```

这可能是由于SSL证书加密没有使用RSA算法导致, 解决方法:
- 编译curl使用OpenSSL模块，不是默认的NSS模块
- 生成证书时使用RSA加密算法

参考:
- [//stackoverflow.com/questions/20969241/curl-58-unable-to-load-client-key-8178/22503032](https://stackoverflow.com/questions/20969241/curl-58-unable-to-load-client-key-8178/22503032)
- [https://bugzilla.redhat.com/show_bug.cgi?id=1051533](https://bugzilla.redhat.com/show_bug.cgi?id=1051533)


## 参考
- [https://github.com/coreos/etcd/tree/master/Documentation](https://github.com/coreos/etcd/tree/master/Documentation)
- [https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md)
- [https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html](https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html)
- [https://coreos.com/os/docs/latest/generate-self-signed-certificates.html](https://coreos.com/os/docs/latest/generate-self-signed-certificates.html)
