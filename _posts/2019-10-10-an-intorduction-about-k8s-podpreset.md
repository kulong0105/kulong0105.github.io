---
title: K8S podpreset介绍
category: K8S
tags:
- K8S
---


## 背景

自K8S版本1.8开始就支持podpreset(但initContainer注入在1.14版本才开始支持)，在使用podpreset后可不需要每个服务配置完全重复的env和secret

<!--more-->

## 使用

根据官方文档总结如下：
- podpreset使用”label selectors“来决定向哪些pod注入信息
- podpreset成功注入信息到pod后，会在pod spec中增加个`annotation: podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>".`
- 可以为特定pod关闭podpreset信息注入，只要在pod spec中增加个`annotation: podpreset.admission.kubernetes.io/exclude: "true"`
- k8s默认没有启动podpreset特性，需要做两处修改enable起来：
    - 在apiserver的配置选项--runtime-config中添加settings.k8s.io/v1alpha1=true
    - 在apiserver的配置选项--enable-admission-plugins中添加PodPreset


## 验证

- 修改apiserver启动配置

```
[root@skyaxe-app-1 podpreset-test]# cat /etc/kubernetes/manifests/kube-apiserver.yaml  | head -20
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.50.111
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction,PodPreset              <======
    - --runtime-config=settings.k8s.io/v1alpha1=true                    <======
    - --enable-bootstrap-token-auth=true
[root@skyaxe-app-1 podpreset]#
```

- 编写podpreset配置文件：

```
[root@skyaxe-app-1 podpreset-test]# cat podpreset.yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: skydiscovery-podpreset
  namespace: skydiscovery                                <======
spec:
  selector:
    matchLabels:
      iluvartar.ai/podpreset: "true"                     <======
  env:
    - name: DB_PORT
      value: "6379"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
[root@skyaxe-app-1 podpreset]#
```

- 编写deployment配置文件

```
[root@skyaxe-app-1 podpreset-test]# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: skydiscovery-test
  namespace: skydiscovery
  labels:
    k8s-app: skydiscovery-test
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      k8s-app: skydiscovery-test
  template:
    metadata:
      labels:
        k8s-app: skydiscovery-test
        iluvartar.ai/podpreset: "true"                   <======
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      nodeSelector:
        node-role.kubernetes.io/master: ""
      containers:
      - name: centos-test
        image: centos
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo hello; sleep 10;done"]
        ports:
        - containerPort: 27017
          name: centos-port
          protocol: TCP
  [root@skyaxe-app-1 podpreset]#
```

- apply编写的podpreset和deployment文件

```
[root@skyaxe-app-1 podpreset-test]# kubectl  apply -f deployment.yaml
[root@skyaxe-app-1 podpreset-test]# kubectl  apply -f podpreset.yaml
```

- 查看pod信息:

```
[root@skyaxe-app-1 podpreset]# kdes pod skydiscovery-test-599f4f75d8-7nnfw -n skydiscovery
Name:           skydiscovery-test-599f4f75d8-7nnfw
Namespace:      skydiscovery
Priority:       0
Node:           skyaxe-app-2.localdomain/192.168.50.112
Start Time:     Mon, 02 Sep 2019 16:35:14 +0800
Labels:         iluvartar.ai/podpreset=true
                k8s-app=skydiscovery-test
                pod-template-hash=599f4f75d8
Annotations:    podpreset.admission.kubernetes.io/podpreset-skydiscovery-podpreset: 33937                <======
Status:         Running
IP:             198.168.205.9
Controlled By:  ReplicaSet/skydiscovery-test-599f4f75d8
...
    Environment:
      DB_PORT:  6379
    Mounts:
      /cache from cache-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-h64bm (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  cache-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
```


## 参考
- https://kubernetes.io/docs/concepts/workloads/pods/podpreset/
- https://kubernetes.io/docs/tasks/inject-data-application/podpreset/
