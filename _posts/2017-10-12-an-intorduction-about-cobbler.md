---
title: Cobbler自动化批量安装操作系统
category: linux
tags:
- linux
---

## 介绍

- Cobbler是一个通过网络安装linux的工具，使用python开发，小巧轻便
- Cobbler可以使用简单的命令完成PXE网络安装环境的配置
- Cobbler可以管理DHCP、DNS、TFTP、RSYNC以及YUM仓库、构造系统ISO镜像
- Cobbler支持命令行管理，WEB界面管理，还提供了API接口，可以二次开发使用

本文将详细介绍如何使用docker化的Cobbler来自动化批量安装操作系统

<!--more-->

## Docker-Cobbler 使用方法

访问 [https://github.com/kulong0105/docker-cobbler](https://github.com/kulong0105/docker-cobbler)

