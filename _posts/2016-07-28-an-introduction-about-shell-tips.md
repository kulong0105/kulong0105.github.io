---
title: Shell Tips
category: linux
tags:
- shell
- tips
---

## 介绍

1. shell多行注释

* 正式表达式

```bash
:2,5s/^/#/      (添加注释符#从第2行到第5行)
:2,5s/^#//      (去掉注释符#从第2行到第5行)
```

* here文档

```bash
:<<\EOF
…
注释内容
…
EOF
```

说明：由于”：”是空命令，把输入重定义到命令”：”后，就相当于注释了。

* 循环实现

```bash
while ：
do
	break
	注释内容
done 
```

说明：如果注释内容中含有“done”，会造成意外循环结束。

