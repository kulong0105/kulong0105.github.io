---
title: K8S安装包构建介绍
category: K8S
tags:
- K8S
---


## 背景

由于K8S官方提供的安装包可能不满足生产环境需求，以及对K8S的扩展进行开发的需求，需要对K8S从源代码去构建安装包。为了能够部署K8S集群，需要如下安装包：

- 镜像
    - k8s.gcr.io/kube-controller-manager-amd64
    - k8s.gcr.io/kube-scheduler-amd64
    - k8s.gcr.io/kube-proxy-amd64
    - k8s.gcr.io/kube-apiserver-amd64
    - k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64
    - k8s.gcr.io/k8s-dns-sidecar-amd64
    - k8s.gcr.io/k8s-dns-kube-dns-amd64
    - k8s.gcr.io/coredns
    - k8s.gcr.io/pause-amd64

- rpm包
    - kubelet
    - kubeadm
    - kubectl
    - cri-tools
    - kubernetes-cni

<!--more-->


## 构建安装包

### 下载源码

```
[root@k1 ~]# mkdir k8s_build
[root@k1 ~]# cd k8s_build/
[root@k1 k8s_build]# git clone https://github.com/kubernetes/kubernetes
```

### 镜像构建
K8S集群所需要的相关镜像，无法通过一次编译全部构建，主要分为如下几类：
- kube-* 相关镜像
- pause 镜像
- dns镜像 ( V1.13版本之前：k8s-dns， V1.13版本之后：coredns）

#### kube-*相关镜像

K8S支持两种方式构建：Go环境和Docker环境。为了方便，采用Docker环境进行构建：
```
[root@k1 k8s_build]# cd kubernetes
[root@k1 kubernetes]# KUBE_BUILD_HYPERKUBE=n KUBE_BUILD_CONFORMANCE=n make quick-release-images
+++ [0327 18:15:36] Verifying Prerequisites....
+++ [0327 18:15:36] Building Docker image kube-build:build-118732ff5e-5-v1.12.0-1
+++ [0327 18:15:41] Creating data container kube-build-data-118732ff5e-5-v1.12.0-1
+++ [0327 18:17:03] Syncing sources to container
+++ [0327 18:17:11] Running build command...
+++ [0327 18:17:19] Building go targets for linux/amd64:
    ./vendor/k8s.io/code-generator/cmd/deepcopy-gen
+++ [0327 18:17:28] Building go targets for linux/amd64:
    ./vendor/k8s.io/code-generator/cmd/defaulter-gen
+++ [0327 18:17:34] Building go targets for linux/amd64:
    ./vendor/k8s.io/code-generator/cmd/conversion-gen
+++ [0327 18:17:44] Building go targets for linux/amd64:
    ./vendor/k8s.io/kube-openapi/cmd/openapi-gen
+++ [0327 18:17:53] Building go targets for linux/amd64:
    ./vendor/github.com/jteeuwen/go-bindata/go-bindata
+++ [0327 18:17:54] Building go targets for linux/amd64:
    cmd/cloud-controller-manager
    cmd/kube-apiserver
    cmd/kube-controller-manager
    cmd/kube-scheduler
    cmd/kube-proxy
+++ [0327 18:19:22] Syncing out of container
+++ [0327 18:19:37] Building images: linux-amd64
+++ [0327 18:19:37] Starting docker build for image: cloud-controller-manager-amd64
+++ [0327 18:19:37] Starting docker build for image: kube-apiserver-amd64
+++ [0327 18:19:37] Starting docker build for image: kube-controller-manager-amd64
+++ [0327 18:19:37] Starting docker build for image: kube-scheduler-amd64
+++ [0327 18:19:37] Starting docker build for image: kube-proxy-amd64
+++ [0327 18:20:00] Deleting docker image k8s.gcr.io/kube-scheduler:v1.15.0-alpha.0.1209_897d62ace79a8f-dirty
+++ [0327 18:20:00] Deleting docker image k8s.gcr.io/kube-controller-manager:v1.15.0-alpha.0.1209_897d62ace79a8f-dirty
+++ [0327 18:20:01] Deleting docker image k8s.gcr.io/kube-apiserver:v1.15.0-alpha.0.1209_897d62ace79a8f-dirty
+++ [0327 18:20:01] Deleting docker image k8s.gcr.io/cloud-controller-manager:v1.15.0-alpha.0.1209_897d62ace79a8f-dirty
+++ [0327 18:20:03] Deleting docker image k8s.gcr.io/kube-proxy:v1.15.0-alpha.0.1209_897d62ace79a8f-dirty
+++ [0327 18:20:03] Docker builds done
[root@k1 kubernetes]# ll _output/release-images/amd64/
total 665460
drwxr-xr-x 2 root root       151 Mar 27 18:20 .
drwxr-xr-x 3 root root        19 Mar 27 18:19 ..
-rw-r--r-- 2 root root 144085504 Mar 27 18:20 cloud-controller-manager.tar
-rw-r--r-- 2 root root 211171328 Mar 27 18:20 kube-apiserver.tar
-rw-r--r-- 2 root root 159213056 Mar 27 18:20 kube-controller-manager.tar
-rw-r--r-- 2 root root  83892736 Mar 27 18:20 kube-proxy.tar
-rw-r--r-- 2 root root  83055616 Mar 27 18:20 kube-scheduler.tar
[root@k1 kubernetes]#
```
可以看出编译完成之后，相关镜像tar包存放在_output/release-images/amd64目录下

#### pause镜像

```
[root@k1 kubernetes]# cd build/pause/
[root@k1 pause]# make container
mkdir -p bin
docker run --rm -u $(id -u):$(id -g) -v $(pwd):/build \
	k8s.gcr.io/kube-cross:v1.12.0-1 \
	/bin/bash -c "\
		cd /build && \
		x86_64-linux-gnu-gcc -Os -Wall -Werror -static -DVERSION=v3.1- -o bin/pause-amd64 pause.c && \
		x86_64-linux-gnu-strip bin/pause-amd64"
docker build --pull -t staging-k8s.gcr.io/pause-amd64:3.1 --build-arg ARCH=amd64 .
Sending build context to Docker daemon  752.1kB
Step 1/4 : FROM scratch
 --->
Step 2/4 : ARG ARCH
 ---> Running in 734be3d0065d
Removing intermediate container 734be3d0065d
 ---> ff57b4a98838
Step 3/4 : ADD bin/pause-${ARCH} /pause
 ---> f765bbc1b361
Step 4/4 : ENTRYPOINT ["/pause"]
 ---> Running in f8cd08582fb4
Removing intermediate container f8cd08582fb4
 ---> 4165d23b7fcd
Successfully built 4165d23b7fcd
Successfully tagged staging-k8s.gcr.io/pause-amd64:3.1
touch .container-amd64
[root@k1 pause]# docker images | grep pause
staging-k8s.gcr.io/pause-amd64                   3.1                            4165d23b7fcd        About a minute ago   738kB
[root@k1 pause]#
```

#### dns镜像

- kube-dns镜像
```
[root@k1 pause]# cd ~/k8s_build/
[root@k1 k8s_build]# git clone  https://github.com/kubernetes/dns.git
[root@k1 k8s_build]# cd dns/
[root@k1 dns]# make containers
building : bin/amd64/dnsmasq-nanny
1.11-alpine: Pulling from library/golang
Digest: sha256:c895ad3b69f2d36805034c3237d39ccd5b761cf3e59ba623616513b44ece413b
Status: Image is up to date for golang:1.11-alpine
make[1]: Entering directory `/root/dns/images'
make[2]: Entering directory `/root/dns/images'
make[3]: Entering directory `/root/dns/images/dnsmasq'
make[3]: Nothing to be done for `containers'.
make[3]: Leaving directory `/root/dns/images/dnsmasq'
make[2]: Leaving directory `/root/dns/images'
make[1]: Leaving directory `/root/dns/images'
container: bin/amd64/dnsmasq-nanny (staging-k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64)
3.8: Pulling from library/alpine
Digest: sha256:a4d41fa0d6bb5b1194189bab4234b1f2abfabb4728bda295f5c53d89766aa046
Status: Image is up to date for alpine:3.8
building : bin/amd64/kube-dns
1.11-alpine: Pulling from library/golang
Digest: sha256:c895ad3b69f2d36805034c3237d39ccd5b761cf3e59ba623616513b44ece413b
Status: Image is up to date for golang:1.11-alpine
container: bin/amd64/kube-dns (staging-k8s.gcr.io/k8s-dns-kube-dns-amd64)
3.8: Pulling from library/alpine
Digest: sha256:a4d41fa0d6bb5b1194189bab4234b1f2abfabb4728bda295f5c53d89766aa046
Status: Image is up to date for alpine:3.8
building : bin/amd64/node-cache
1.11-alpine: Pulling from library/golang
Digest: sha256:c895ad3b69f2d36805034c3237d39ccd5b761cf3e59ba623616513b44ece413b
Status: Image is up to date for golang:1.11-alpine
container: bin/amd64/node-cache (staging-k8s.gcr.io/k8s-dns-node-cache-amd64)
3.8: Pulling from library/alpine
Digest: sha256:a4d41fa0d6bb5b1194189bab4234b1f2abfabb4728bda295f5c53d89766aa046
Status: Image is up to date for alpine:3.8
building : bin/amd64/sidecar
1.11-alpine: Pulling from library/golang
Digest: sha256:c895ad3b69f2d36805034c3237d39ccd5b761cf3e59ba623616513b44ece413b
Status: Image is up to date for golang:1.11-alpine
container: bin/amd64/sidecar (staging-k8s.gcr.io/k8s-dns-sidecar-amd64)
3.8: Pulling from library/alpine
Digest: sha256:a4d41fa0d6bb5b1194189bab4234b1f2abfabb4728bda295f5c53d89766aa046
Status: Image is up to date for alpine:3.8
[root@k1 dns]# docker images | grep dns
staging-k8s.gcr.io/k8s-dns-sidecar-amd64         1.15.1-4-gdf551b6-dirty        ca8f631e1964        2 days ago          41.4MB
staging-k8s.gcr.io/k8s-dns-node-cache-amd64      1.15.1-4-gdf551b6-dirty        58918470af03        2 days ago          77.8MB
staging-k8s.gcr.io/k8s-dns-kube-dns-amd64        1.15.1-4-gdf551b6-dirty        26e246071c3a        2 days ago          49.1MB
staging-k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64   1.15.1-4-gdf551b6-dirty        bf75b8e494fb        2 days ago          39.8MB
staging-k8s.gcr.io/k8s-dns-dnsmasq-amd64         1.15.1-4-gdf551b6-dirty        fae11611acf0        2 days ago          4.77MB
[root@k1 dns]#
```

- coredns镜像

由于coredns不是通过docker方式编译，所以需要预先安装好go编译环境(https://golang.org/doc/install#install)
```
[root@k1 dns]# cd ~/k8s_build/
[root@k1 k8s_build]# git clone https://github.com/coredns/coredns.git
[root@k1 k8s_build]# cd coredns
[root@k1 coredns]# make
** presubmit/context
** presubmit/filename-hyphen
** presubmit/import-testing
go: finding github.com/kr/pty v1.1.1
go: finding honnef.co/go/tools v0.0.0-20190102054323-c2f93a96b099
go: finding gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405
go: finding gopkg.in/resty.v1 v1.12.0
go: downloading github.com/mholt/caddy v0.11.5
go: downloading gopkg.in/DataDog/dd-trace-go.v0 v0.6.1
...
...
...
k8s.io/client-go/kubernetes/typed/storage/v1alpha1
k8s.io/client-go/kubernetes/typed/storage/v1beta1
k8s.io/client-go/kubernetes
github.com/DataDog/dd-trace-go/tracer
github.com/coredns/coredns/plugin/kubernetes
gopkg.in/DataDog/dd-trace-go.v0/opentracing
github.com/coredns/coredns/plugin/trace
github.com/coredns/coredns/plugin/federation
github.com/coredns/coredns/core/plugin
github.com/coredns/coredns
[root@k1 coredns]# ls -al coredns
-rwxr-xr-x 1 root root 39684864 Mar 27 22:52 coredns
[root@k1 coredns]# docker build -t coredns .
Sending build context to Docker daemon  39.91MB
Step 1/8 : FROM debian:stable-slim
 ---> 65d9110be43c
Step 2/8 : RUN apt-get update && apt-get -uy upgrade
 ---> Running in f5def3dbd58f
...
...
...
Step 4/8 : FROM scratch
 --->
Step 5/8 : COPY --from=0 /etc/ssl/certs /etc/ssl/certs
 ---> 2fb88685fee5
Step 6/8 : ADD coredns /coredns
 ---> 946bf4f45fcb
Step 7/8 : EXPOSE 53 53/udp
 ---> Running in d689a06d70f7
Removing intermediate container d689a06d70f7
 ---> fa8ba6fd1efe
Step 8/8 : ENTRYPOINT ["/coredns"]
 ---> Running in 7c4a5820d87b
Removing intermediate container 7c4a5820d87b
 ---> 6c5dc7b306ab
Successfully built 6c5dc7b306ab
Successfully tagged coredns:latest
[root@k1 coredns]# docker images  | grep coredns
coredns                                          latest                         6c5dc7b306ab        2 minutes ago       39.9MB
[root@k1 coredns]#
```


## RPM构建

### 构建kubectl / kubelet / kubeadm

```
[root@k1 coredns]# cd ~/k8s_build/kubernetes
[root@k1 kubernetes]# ./build/run.sh make kubectl kubelet kubeadm KUBE_BUILD_PLATFORMS=linux/amd64
+++ [0327 23:16:08] Verifying Prerequisites....
+++ [0327 23:16:09] Building Docker image kube-build:build-118732ff5e-5-v1.12.0-1
+++ [0327 23:16:13] Keeping container optimistic_rubin
+++ [0327 23:16:13] Keeping container optimistic_rubin
+++ [0327 23:16:13] Keeping container optimistic_rubin
+++ [0327 23:16:13] Keeping image kube-build:build-118732ff5e-5-v1.12.0-1
+++ [0327 23:16:13] Creating data container kube-build-data-118732ff5e-5-v1.12.0-1
+++ [0327 23:17:36] Syncing sources to container
+++ [0327 23:17:36] Stopping any currently running rsyncd container
+++ [0327 23:17:36] Starting rsyncd container
+++ [0327 23:17:37] Running rsync
+++ [0327 23:17:57] Stopping any currently running rsyncd container
+++ [0327 23:17:58] Output from this container will be rsynced out upon completion. Set KUBE_RUN_COPY_OUTPUT=n to disable.
+++ [0327 23:17:58] Running build command...
Go version: go version go1.12 linux/amd64
+++ [0327 23:18:08] Building go targets for linux/amd64:
    ./vendor/k8s.io/code-generator/cmd/deepcopy-gen
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
+++ [0327 23:18:10] Placing binaries
I0327 23:18:11.223945    2118 parse.go:327] importPackage k8s.io/kubernetes/cmd/cloud-controller-manager/app/apis/config
I0327 23:18:11.224003    2118 parse.go:287] addDir k8s.io/kubernetes/cmd/cloud-controller-manager/app/apis/config
...
...
...
Generated bindata file : test/e2e/generated/bindata.go has 9572 test/e2e/generated/bindata.go lines of lovely automated artifacts
No changes in generated bindata file: pkg/kubectl/generated/bindata.go
Go version: go version go1.12 linux/amd64
+++ [0327 23:18:44] Building go targets for linux/amd64:
    cmd/kubectl
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
+++ [0327 23:19:09] Placing binaries
Go version: go version go1.12 linux/amd64
+++ [0327 23:19:10] Building go targets for linux/amd64:
    cmd/kubelet
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
+++ [0327 23:20:06] Placing binaries
Go version: go version go1.12 linux/amd64
+++ [0327 23:20:08] Building go targets for linux/amd64:
    cmd/kubeadm
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
+++ [0327 23:20:19] Placing binaries
+++ [0327 23:20:22] Syncing out of container
+++ [0327 23:20:22] Stopping any currently running rsyncd container
+++ [0327 23:20:22] Starting rsyncd container
+++ [0327 23:20:24] Running rsync
+++ [0327 23:20:26] Stopping any currently running rsyncd container
[root@k1 kubernetes]# tree _output/
_output/
├── dockerized
│   ├── bin
│   │   └── linux
│   │       └── amd64
│   │           ├── conversion-gen
│   │           ├── deepcopy-gen
│   │           ├── defaulter-gen
│   │           ├── go2make
│   │           ├── go-bindata
│   │           ├── kubeadm
│   │           ├── kubectl
│   │           ├── kubelet
│   │           └── openapi-gen
│   └── go
└── images
    └── kube-build:build-118732ff5e-5-v1.12.0-1
        ├── Dockerfile
        ├── localtime
        ├── rsyncd.password
        └── rsyncd.sh

7 directories, 13 files
[root@k1 kubernetes]# yum install -y rpm-build rpmdevtools createrepo
[root@k1 kubernetes]# rpmdev-setuptree
[root@k1 kubernetes]# cp -a _output/dockerized/bin/linux/amd64/*  ~/rpmbuild/BUILD/
[root@k1 kubernetes]# cp -a build/rpms/* ~/rpmbuild/BUILD/
[root@k1 kubernetes]# sed -i -e "s#{\(kubectl.*\)}#\1#" ~/rpmbuild/BUILD/kubectl.spec
[root@k1 kubernetes]# sed -i -e "s#{\(kubelet.*\)}#\1#" ~/rpmbuild/BUILD/kubelet.spec
[root@k1 kubernetes]# sed -i -e "s#\ {\(.*kubeadm.*\)}# \1#" -e "s#{kubelet.env}#kubelet.env# " ~/rpmbuild/BUILD/kubeadm.spec
[root@k1 kubernetes]# rpmbuild -bb  ~/rpmbuild/BUILD/kubectl.spec
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.7bdlKF
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64
++ dirname /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64/usr/bin
+ install -p -m 755 -t /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64/usr/bin kubectl
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
Processing files: kubectl-OVERRIDE_THIS-00.x86_64
Provides: kubectl = OVERRIDE_THIS-00 kubectl(x86-64) = OVERRIDE_THIS-00
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/kubectl-OVERRIDE_THIS-00.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.7zQ7y3
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kubectl-OVERRIDE_THIS-00.x86_64
+ exit 0
[root@k1 kubernetes]# rpmbuild -bb  ~/rpmbuild/BUILD/kubelet.spec
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.9MwqnC
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64
++ dirname /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64/usr/bin
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64/etc/systemd/system/
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64/etc/kubernetes/manifests/
+ install -p -m 755 -t /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64/usr/bin kubelet
+ install -p -m 644 -t /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64/etc/systemd/system/ kubelet.service
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
Processing files: kubelet-OVERRIDE_THIS-00.x86_64
Provides: kubelet = OVERRIDE_THIS-00 kubelet(x86-64) = OVERRIDE_THIS-00
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Requires: libc.so.6()(64bit) libc.so.6(GLIBC_2.2.5)(64bit) libdl.so.2()(64bit) libdl.so.2(GLIBC_2.2.5)(64bit) libpthread.so.0()(64bit) libpthread.so.0(GLIBC_2.2.5)(64bit) libpthread.so.0(GLIBC_2.3.2)(64bit) rtld(GNU_HASH)
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/kubelet-OVERRIDE_THIS-00.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.bySFmD
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kubelet-OVERRIDE_THIS-00.x86_64
+ exit 0
[root@k1 kubernetes]# rpmbuild -bb  ~/rpmbuild/BUILD/kubeadm.spec
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.uV4mbG
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64
++ dirname /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/bin
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/etc/systemd/system/
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/etc/systemd/system/kubelet.service.d/
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/etc/sysconfig/
+ install -p -m 755 -t /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/bin kubeadm
+ install -p -m 644 -t /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/etc/systemd/system/kubelet.service.d/ 10-kubeadm.conf
+ install -p -m 644 -T kubelet.env /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/etc/sysconfig/kubelet
+ mkdir -p /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/libexec/modules-load.d
+ mkdir -p /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/lib/sysctl.d/
+ install -p -m 0644 -t /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/libexec/modules-load.d/ kubeadm.conf
+ install -p -m 0644 -t /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64/usr/lib/sysctl.d/ 50-kubeadm.conf
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
Processing files: kubeadm-OVERRIDE_THIS-00.x86_64
Provides: kubeadm = OVERRIDE_THIS-00 kubeadm(x86-64) = OVERRIDE_THIS-00
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/kubeadm-OVERRIDE_THIS-00.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.jxLoaS
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kubeadm-OVERRIDE_THIS-00.x86_64
+ exit 0
[root@k1 kubernetes]# createrepo  ~/rpmbuild/RPMS/x86_64/
Spawning worker 0 with 1 pkgs
Spawning worker 1 with 1 pkgs
Spawning worker 2 with 1 pkgs
Spawning worker 3 with 0 pkgs
Spawning worker 4 with 0 pkgs
Spawning worker 5 with 0 pkgs
Spawning worker 6 with 0 pkgs
Spawning worker 7 with 0 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@k1 x86_64]# ll ~/rpmbuild/RPMS/x86_64/
total 42160
drwxr-xr-x 3 root root      151 Mar 28 11:28 .
drwxr-xr-x 3 root root       20 Mar 28 11:25 ..
-rw-r--r-- 1 root root  9136564 Mar 28 11:26 kubeadm-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root  9937828 Mar 28 11:25 kubectl-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root 24088072 Mar 28 11:26 kubelet-OVERRIDE_THIS-00.x86_64.rpm
drwxr-xr-x 2 root root     4096 Mar 28 11:28 repodata
[root@k1 kubernetes]#
```

### 构建kubernetes-cni / crictl-tools

由于几乎不会对这两个包的源代码进行修改，就不介绍如何从源代码去编译这两个包，这里仅仅介绍如何获取对应的二进制包以及如何打成rpm包:  (根据需求，可下载不同版本的二进制文件）
```
[root@k1 kubernetes]# wget -q https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.11.1/crictl-v1.11.1-linux-amd64.tar.gz -O ~/rpmbuild/BUILD/cri_tools.tgz
[root@k1 kubernetes]# wget -q https://dl.k8s.io/network-plugins/cni-plugins-amd64-v0.7.5.tgz -O ~/rpmbuild/BUILD/kubernetes_cni.tgz
[root@k1 kubernetes]# sed -i -e "s/{cri_tools.tgz}/cri_tools.tgz/"  cri-tools.spec
[root@k1 kubernetes]# sed -i -e "s/{kubernetes_cni.tgz}/kubernetes_cni.tgz/"  kubernetes-cni.spec
[root@k1 kubernetes]# rpmbuild  -bb  ~/rpmbuild/BUILD/cri-tools.spec
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.TV3JWQ
+ umask 022
+ cd /root/rpmbuild/BUILD
+ tar -xzf cri_tools.tgz
+ exit 0
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.ZrgrKL
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64
++ dirname /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64
+ install -m 755 -d /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64/usr/bin
+ install -p -m 755 -t /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64/usr/bin crictl
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
Processing files: cri-tools-OVERRIDE_THIS-00.x86_64
Provides: cri-tools = OVERRIDE_THIS-00 cri-tools(x86-64) = OVERRIDE_THIS-00
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/cri-tools-OVERRIDE_THIS-00.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.Be3luI
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/cri-tools-OVERRIDE_THIS-00.x86_64
+ exit 0
[root@k1 kubernetes]# rpmbuild  -bb  ~/rpmbuild/BUILD/kubernetes-cni.spec
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.aAmM2l
+ umask 022
+ cd /root/rpmbuild/BUILD
+ mkdir -p ./bin
+ tar -C ./bin -xz -f kubernetes_cni.tgz
+ exit 0
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.Ya2zKF
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64
++ dirname /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64/etc/cni/net.d/
+ install -m 755 -d /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64/opt/cni
+ mv bin/ /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64/opt/cni/
+ '[' '%{buildarch}' = noarch ']'
+ QA_CHECK_RPATHS=1
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-rpaths
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
+ /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
Processing files: kubernetes-cni-OVERRIDE_THIS-00.x86_64
Provides: kubernetes-cni = OVERRIDE_THIS-00 kubernetes-cni(x86-64) = OVERRIDE_THIS-00
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Checking for unpackaged file(s): /usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64
Wrote: /root/rpmbuild/RPMS/x86_64/kubernetes-cni-OVERRIDE_THIS-00.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.6sg1Tx
+ umask 022
+ cd /root/rpmbuild/BUILD
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kubernetes-cni-OVERRIDE_THIS-00.x86_64
+ exit 0
[root@k1 kubernetes]# createrepo  /root/rpmbuild/RPMS/x86_64/
Spawning worker 0 with 1 pkgs
Spawning worker 1 with 1 pkgs
Spawning worker 2 with 1 pkgs
Spawning worker 3 with 1 pkgs
Spawning worker 4 with 1 pkgs
Spawning worker 5 with 0 pkgs
Spawning worker 6 with 0 pkgs
Spawning worker 7 with 0 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@k1 kubernetes]# ll /root/rpmbuild/RPMS/x86_64/
total 56824
drwxr-xr-x 3 root root      246 Mar 28 17:29 .
drwxr-xr-x 3 root root       20 Mar 28 11:25 ..
-rw-r--r-- 1 root root  4376184 Mar 28 17:29 cri-tools-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root  9136564 Mar 28 11:26 kubeadm-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root  9937828 Mar 28 11:25 kubectl-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root 24088072 Mar 28 11:26 kubelet-OVERRIDE_THIS-00.x86_64.rpm
-rw-r--r-- 1 root root 10634128 Mar 28 17:29 kubernetes-cni-OVERRIDE_THIS-00.x86_64.rpm
drwxr-xr-x 2 root root     4096 Mar 28 17:29 repodata
[root@k1 kubernetes]#
```


## 总结

介绍了部署K8S集群所需要哪些安装包，以及如何从源码去构建对应的镜像文件和rpm包
构建出来的镜像文件和rpm无法直接构建K8S集群，需要根据编译的版本去修改对应的镜像tag和rpm包版本 (由于使用kubeadm去部署K8S集群时，其会绑定到固定版本的镜像)

## 参考

- https://github.com/kubernetes/kubernetes/blob/master/build/README.md
- https://github.com/coredns/coredns
- https://github.com/kubernetes/dns
- https://github.com/kubernetes-sigs/cri-tools
- https://github.com/containernetworking/cni
- https://github.com/kubernetes/release\

