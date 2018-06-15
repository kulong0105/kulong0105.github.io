---
title: 自动化部署高可用Kubernetes集群
category: K8S
tags:
- K8S
---

## 介绍

本文将介绍如何使用kubeadm工具来完成高可用的Kubernetes集群部署，主要包括：

- 使用kubeadm工具来部署Kubernetes
- 使用单独的TLS-etcd集群来保存集群数据
- 使用keepalived来保证集群的高可用
- 使用nginx来完成负载均衡

<!--more-->

## 自动化部署

访问 [https://github.com/kulong0105/kubeadm-ha](https://github.com/kulong0105/kubeadm-ha)

