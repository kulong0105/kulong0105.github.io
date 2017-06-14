---
title: 基于GNU GLOBAL浏览代码
category: linux
tags:
- linux
- global
---

## 介绍

本文将简述如何基于GNU GLOBAL来达到使用Web浏览代码。


### 安装Apache服务器

```
$ yum install httpd
```


### 安装GNU Global软件包

```
$ yum install global
```

<!--more-->


### 生成GTAGS文件及HTML目录  

```
$ cd $source_dir
$ gtags -v 
$ htags -sofFnvaIh #或者 htags --suggest2
```


### 修改/etc/httpd/conf/httpd.conf配置文件

* 修改DocumentRoot所指向的Web目录，将其改为htags命令生成的HTML目录

```
DocumentRoot "/home/renyl/repo/irqbalance/HTML"
```

* 修改"ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"" 变为"ScriptAlias /cgi-bin/ <htags命令生成的cgi-bin目录>"

```
ScriptAlias /cgi-bin/ "/home/renyl/repo/irqbalance/HTML/cgi-bin"
```

* 把配置文件中原有的/var/www/cgi-bin/都替换为<htags命令生成的HTML目录>

```
<Directory "/home/renyl/repo/irqbalance/HTML">
	AllowOverride ALL
	Options None
	Require all granted
</Directory>
```

* 添加如下语句:（gtags命令生成的GTAGS、GPATH、GRTAGS文件所在目录）

```
<Directory "/home/renyl/repo/irqbalance">
	Options +ExecCGI
	AddHandler cgi-script .cgi
	Require all granted
</Directory>
```


### 启动Apache服务器

```
$ sudo httpd -k start
```


### 访问

打开浏览器，访问 localhost:80

注： 如果不能正常访问，可以查看/var/log/下面的log文件，确认是否是权限问题导致。


### 效果

可以参考： http://www.tamacom.com/tour/kernel/linux/

