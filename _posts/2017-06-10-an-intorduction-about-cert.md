---
title: OpenSSL和CFSSL使用简介
category: linux
tags:
- linux
---

## 介绍

- OpenSSL是开源的SSL（Secure Sockets Layer）实现，SSL 是一个安全协议，为基于 TCP 的应用层提供安全链接，其目标是保证两个应用间通信的保密性和可靠性
- CFSSL 是 CloudFlare 公司的一个 PKI 工具包，提供了命令行工具和一个 HTTP API 服务器用于签名、验证和捆绑 SSL 证书

本文将对OpenSSL和CFSSL的使用进行介绍

<!--more-->


## OpenSSL

OpenSSL主要提供如下功能:

- 对称加密算法（如DES）
- 非对称加密算法（如RSA）
- 信息摘要算法（如MD5）
- 证书管理(如CA）


### 配置文件位置
/etc/pki/tls/openssl.cnf

### 对称加密

1) 加密
```
$ openssl enc -aes-256-ecb -a -in ./test1 -out ./test2 -k kulong520
```

2) 解密
```
$ openssl enc -aes-256-ecb -d -a -in ./test2 -out ./test3 -k kulong520
```
- -a 表示使用base64
- -d 表示解密
- -k 表示加密密码
- test1表示原始文件，test2表示加密后的文件，test3表示解密后的文件（与test1应该一模一样）


### 非对称加密

1) 生成私钥
```
$ openssl genrsa -out allen.key 2048
```

2) 提取公钥
```
$ openssl rsa -in allen.key -pubout > allen.public
```

### 信息摘要

```
$ openssl passwd -1 "rootroot"
$1$GXlfA2B0$rU/d.JX6qjNfudAIkya8M.
```

注： 生成的字符串可以放到/etc/passwd文件中，直接设置用户的密码


### 证书管理

1) 生成CA证书
- 生成私钥

```
$ mkdir -p demoCA/{private,certs,crl,newcerts}
$ openssl genrsa -out demoCA/private/cakey.pem 2048`
```

- 生成自签证书

```
$ openssl req -new -nodes -x509 -key demoCA/private/cakey.pem -days 3650 -out demoCA/cacert.pem \
        -subj "/C=CN/ST=Jiangsu/L=Nanjing/O=SkyData/OU=platform/CN=$IP/emailAddress=yilong.ren@sky-data.cn"
```

- 创建用于签名证书的相关文件

```
$ touch demoCA/{index.txt,serial,crlnumber}
$ echo "01" > demoCA/serial`
```

参数说明：
* req: X.509 Certificate Signing Request (CSR) Management.
* -x509: this option outputs a self signed certificate instead of a certificate reques
* -nodes: if this option is specified then if a private key is created it will not be encrypted.
* CN: Common Name
* C: Country
* ST: State
* L: Locality
* O: Organization Name
* OU: Organization Unit Name
* $IP: replace with IP Addr



2) 生成CSR和CA签名
- 生成私钥

```
$ mkdir -p mycrt
$ openssl genrsa -out mycrt/allen.key 2048`
```

- 申请CSR

```
$ openssl req -new -key mycrt/allen.key -out mycrt/allen.csr \
        -subj "/C=CN/ST=Jiangsu/L=Nanjing/O=SkyData/OU=platform/CN=$IP/emailAddress=yilong.ren@sky-data.cn"`
```

- 使用CA签署

```
$ openssl ca -in mycrt/allen.csr -out mycrt/allen.crt -days 3650 -batch`
```

- 查看证书信息

```
$ openssl x509 -noout -text -in mycrt/allen.crt
```

## CFSSL

生成证书步骤如下：

1) ca 配置文件
```
$ cat json/ca-config.json
{
    "signing": {
        "default": {
            "expiry": "43800h"
        },
        "profiles": {
            "server": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            },
            "client": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "43800h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

2) ca csr 配置文件
```
$ cat json/ca-csr.json
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Jiangsu",
      "L": "Nanjing",
      "O": "SkyData",
      "OU": "Platform"
    }
  ]
}
```

3) 申请机器的 csr 配置文件
```
$ cat json/my_csr.json
{
  "CN": "${hostname}",
  "hosts": [
    "${hostname}",
    "${node_ip}"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Jiangsu",
      "L": "Nanjing",
      "O": "SkyData",
      "OU": "Platform"
    }
  ]
}
```

4) 生成CA证书

```
$ cfssl gencert -initca json/ca-csr.json  | cfssljson -bare cert_dir/ca
```
注： 会生成ca.csr, ca-key.pem, ca.pem 三个文件


5) 生成证书

```
$ cfssl gencert -ca=$cert_dir/ca.pem \
                -ca-key=$cert_dir/ca-key.pem \
                -config=json/ca-config.json \
                -profile=$profile json/etcd-csr.json | cfssljson -bare $cert_dir/$profile
```
注： $profile需要替换为server,或peer,或client


## 参考
- [http://blog.51cto.com/fengliang/1855917](http://blog.51cto.com/fengliang/1855917)
- [https://www.jianshu.com/p/15b1d935a44b](https://www.jianshu.com/p/15b1d935a44b)
- [https://www.cnblogs.com/littleatp/p/5878763.html](https://www.cnblogs.com/littleatp/p/5878763.html)
- [https://kubernetes.io/cn/docs/concepts/cluster-administration/certificates/](https://kubernetes.io/cn/docs/concepts/cluster-administration/certificates/)
- [https://blog.cloudflare.com/how-to-build-your-own-public-key-infrastructure/](https://blog.cloudflare.com/how-to-build-your-own-public-key-infrastructure/)
- [https://github.com/cloudflare/cfssl/wiki/Creating-a-new-CSR](https://github.com/cloudflare/cfssl/wiki/Creating-a-new-CSR)
