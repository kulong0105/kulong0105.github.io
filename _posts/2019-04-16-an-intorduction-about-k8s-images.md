---
title: K8S部署所需镜像版本分析
category: K8S
tags:
- K8S
---


## 背景

在对K8S版本升级或修改K8S源码重新编译后，使用kubeadm去部署K8S集群时，会拉取其对应的固定版本kube镜像，因此需要确认相关镜像版本：
- 通过kubeadm去部署K8S集群时，kube-controller-manager-amd64 / kube-scheduler-amd64 / kube-proxy-amd64 / kube-apiserver-amd64 这四个镜像版本跟 kubernetes（即kubelet/kubeadmin）版本保持一致，但是 dns 和 pause 镜像版本跟 kubernetes 版本没有直接关联
- 关于如何确认对应的dns(k8s v1.13之前使用kube-dns； k8s v1.13版本之后使用coredns）和pause镜像版本，是确认镜像版本的重点

<!--more-->

## 确认方式

考虑这种方式：在使用kubeadm init 进行初始化，由于这个时候没有对应版本的镜像：
- kube-dns pod会启动失败，此时通过 kubectl describe命令去看 kube-dns pod 获取镜像版本
- 但由于 pause 不会启动相应pod，只会针对相应pod启动pause container，所以不能获取pause镜像版本

### v1.13版本之前

通过查看源代码去获取：切换到对应版本的分支，相关代码在 `cmd/kubeadm/app`目录下 (具体代码就不分析了）
- kube-apiserver/kube-controller-manager/kube-scheduler 这三个pod对应的镜像被定义在 images/images.go 文件
- kube-dns 对应镜像版本为定义在 phases/addons/dns/versions.go
- proxy 对应镜像版本被定义在 phases/addons/proxy/proxy.go （同kube-apiserver版本一致，只是定义位置不同）
- pause 由于是被kubelet使用，被定义在`cmd/kubelet/app/options/container_runtime.go`文件

### v1.13版本之后

- 在v1.13版本之后，kubeadm代码对镜像版本的定义进行了相关调整，可直接通过命令`kubeadm config images list`列出部署K8S所需要的镜像和版本:

```
[root@k1 amd64]# ./kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.14.0
k8s.gcr.io/kube-controller-manager:v1.14.0
k8s.gcr.io/kube-scheduler:v1.14.0
k8s.gcr.io/kube-proxy:v1.14.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
[root@k1 amd64]#
```

- images的版本获取全部放到`cmd/kubeadm/app/images/images.go`文件，所有镜像版本定义存放在`cmd/kubeadm/app/constants/constants.go`文件

