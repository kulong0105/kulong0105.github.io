---
title: K8S kmem泄露分析及解决
category: K8S
tags:
- K8S
---


## 介绍

本文将对K8S kmem泄露进行分析，以及如何解决进行介绍。

<!--more-->


## 背景

K8S官方提供的kubelet rpm包编译时打开了kmem选项，这将导致kmem内存泄露
```
[root@skyaxe-app-1 kubepods]# pwd
/sys/fs/cgroup/memory/kubepods
[root@skyaxe-app-1 kubepods]# cat memory.kmem.slabinfo
slabinfo - version: 2.1
# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail>
[root@skyaxe-app-1 kubepods]#
```

## 解决方案

升级到最新稳定版，并重新编译kubelet rpm时关闭kmem选项


### 验证方法

验证memory.kmem.slabinfo输出"input/output error"
```
[root@skyaxe-app-1 ~]# cat /sys/fs/cgroup/memory/memory.kmem.slabinfo
cat: /sys/fs/cgroup/memory/memory.kmem.slabinfo: Input/output error
[root@skyaxe-app-1 ~]#
```


### 构建kubelet

构建kublet rpm包:
```
# git clone https://github.com/kubernetes/kubernetes.git
# cd kubernetes
# git checkout v1.15.5
# ./build/run.sh make kubelet GOFLAGS="-v -tags=nokmem" KUBE_BUILD_PLATFORMS=linux/amd64
# rpmdev-setuptree
# cp -a _output/dockerized/bin/linux/amd64/kubelet ~/rpmbuild/BUILD/
# sed -i -e "s#{\(kubelet.*\)}#\1#" ~/rpmbuild/BUILD/kubelet.spec
# sed -i -e 's/OVERRIDE_THIS/1.15.5/g' ~/rpmbuild/BUILD/kubelet.spec
# sed -i -e 's/Release: 00/Release: 1/g' ~/rpmbuild/BUILD/kubelet.spec
# rpmbuild -bb ~/rpmbuild/BUILD/kubelet.spec
```

### kmem泄露分析

### 现象1

- 在130环境的应用节点，直接使用docker 启动一个设置了kernel memory的容器，kmem是不会生效的
```
[root@sdx-app-1 ~]# docker run -tid --kernel-memory 100MB centos /bin/bash
WARNING: You specified a kernel memory limit on a kernel older than 4.0. Kernel memory limits are experimental on older kernels, it won't work as expected and can cause your system to be unstable.
070cbd496aeb266071860af633437683a36933906fe1793b60eb0e1e3f2fa441
[root@sdx-app-1 ~]# cat /sys/fs/cgroup/memory/docker/070cbd496aeb266071860af633437683a36933906fe1793b60eb0e1e3f2fa441/memory.kmem.slabinfo
cat: /sys/fs/cgroup/memory/docker/070cbd496aeb266071860af633437683a36933906fe1793b60eb0e1e3f2fa441/memory.kmem.slabinfo: Input/output error
[root@sdx-app-1 ~]# cat /sys/fs/cgroup/memory/docker/070cbd496aeb266071860af633437683a36933906fe1793b60eb0e1e3f2fa441/memory.kmem.limit_in_bytes
9223372036854771712
}
[root@sdx-app-1 ~]#
```

- 在130环境的计算节点(含有GPU)，直接使用docker 启动一个设置了kernel memory的容器，kmem是会生效的
```
[root@sdx-computing-0 ~]# docker run -tid --kernel-memory 100MB centos /bin/bash
WARNING: You specified a kernel memory limit on a kernel older than 4.0. Kernel memory limits are experimental on older kernels, it won't work as expected and can cause your system to be unstable.
20f050c17baad79d67553988ffc712217220ad2ce9318c8f8d79d9d39b5111d1
[root@sdx-computing-0 ~]# cat /sys/fs/cgroup/memory/docker/20f050c17baad79d67553988ffc712217220ad2ce9318c8f8d79d9d39b5111d1/memory.kmem.slabinfo
slabinfo - version: 2.1
# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail>
taskstats              0      0    328   49    4 : tunables    0    0    0 : slabdata      0      0      0
inode_cache          165    165    592   55    8 : tunables    0    0    0 : slabdata      3      3      0
shmem_inode_cache     48     48    680   48    8 : tunables    0    0    0 : slabdata      1      1      0
ovl_inode            432    432    680   48    8 : tunables    0    0    0 : slabdata      9      9      0
avc_xperms_node        0      0     56   73    1 : tunables    0    0    0 : slabdata      0      0      0
...
[root@sdx-computing-0 ~]# cat /sys/fs/cgroup/memory/docker/20f050c17baad79d67553988ffc712217220ad2ce9318c8f8d79d9d39b5111d1/memory.kmem.limit_in_bytes
104857600
[root@sdx-computing-0 ~]
```

### 分析1

- 130环境的docker runtime是runc，其编译时关闭了kmem，所以即使命令行选项中添加“--kernel-memory”，实际上并不会打开kmem功能（新版runc在编译时如果关闭了kmem，当使用--kernel-memory选项启动容器时会直接报错）
- 130环境的docker runtime是nvidia，该runtime是在runc的基础上做了修改，但是在编译时打开了kmem功能，所以在使用“--kernel-memory”选项启动容器时，会打开kmem功能(新版本nvidia runtime已经关闭了kmem）

### 现象2

- 130环境计算节点使用重新构建的kubelet(编译时关闭了kmem），每个pod泄露2个mem cgroup
- 240环境应用节点使用原生k8s提供的kubelet(编译时打开了kmem），每个pod泄露3个mem cgroup

### 分析2

- k8s在创建pod过程中，会先由kubelet创建一个父的cgroup，然后由docker runtime在该父cgroup下创建两个子cgroup(一个是pause，一个是真实的container）
- cgroup具有继承性，子cgroup会集成父cgroup的属性
- 130环境计算节点kubelet虽然关闭了kmem，但是nvidia runtime没有关闭，由于nvidia runtime负责创建2个子cgroup，所以导致每个pod会泄露2个mem cgroup
- 240环境应用节点runc runtime虽然关闭了kmem，但是kubelet没有关闭，由于cgroup的继承性，所以导致每个pod会泄露3个mem cgroup


## 其他

关于判断当前的runtime是否关闭了kmem，通过命令行启动一个容器即可知道，如：
```
[root@sdx-app-1 ~]# docker run -tid --kernel-memory 100MB busybox /bin/bash
WARNING: You specified a kernel memory limit on a kernel older than 4.0. Kernel memory limits are experimental on older kernels, it won't work as expected and can cause your system to be unstable.
1bf52916556cd69ccac29da59f52f7cf8d019f6035e719065989a251bd8c6590
docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "process_linux.go:430: container init caused \"process_linux.go:396: setting cgroup config for procHooks process caused \\\"kernel memory accounting disabled in this runc build\\\"\"": unknown.
[root@sdx-app-1 ~]#
```
说明：
- 当前SDX环境使用的runc和nvidia runtime都是新版本，其在编译时都关闭了kmem特性
- 只有在pod配置了memory limit的时候才打开memory accounting，即kmem

