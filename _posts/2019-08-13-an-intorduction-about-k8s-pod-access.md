---
title: 外部访问K8S Pod介绍
category: K8S
tags:
- K8S
---

## 访问方式

本文将对从外部访问K8S Pod的几种方式进行简单介绍。

<!--more-->

### hostNetwork

pod会使用Host IP，以及在Host上能够使用netstat看到pod listen的端口号。使用场景，如flannel的部署

### hostPort

pod使用Pod IP，在Host上使用netstat看不到pod listen的端口号，其通过iptables的PREROUTING链来实现，访问Host的端口自动转发到Pod的listen的端口号

### NodePort

pod使用Pod IP，在集群的每个节点都listen一个端口号，默认端口范围必须在：30000-32767，可通过apiserver的--service-node-port-range选项进行配置

### LoadBalancer

在公有云上使用，使用4层负载均衡，不提供SSL termination和HTTP routing功能
与Ingress不同，外部请求需要通过kube-proxy转发到具体的Pod，因此效率没有Ingress高

### Ingress

依赖不同的controller，其支持不同的特性(如：HTTP routing, sticky sessions, SSL termination, SSL passthrough, TCP and UDP load balancing）
基于域名进行访问，由Ingress controller直接(不通过service)反代到各个pod里面


## 参考
http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/
