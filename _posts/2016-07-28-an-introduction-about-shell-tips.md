---
title: Shell Tips
category: linux
tags:
- shell
- tips
---

## 介绍

### shell多行注释

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

 <!--more-->


### if语句与&&的区别

```
[renyl@localhost tmp]$ cat a.sh
#!/bin/bash


test1()
{
	if [[ $abc ]]; then
		echo "YES"
	fi
}

test2()
{
	[[ $abc ]] && {
		echo "YES"
	}
}

test1 && echo "test1"
test2 && echo "test2"

[renyl@localhost tmp]$ ./a.sh
test1
[renyl@localhost tmp]$

说明： if语句作为函数的返回值，总是返回真。而&&这种方式则使用判断语句的返回值作为函数的返回值。

```

### while 语句的特定

```
ips="192.168.10.1
192.168.10.2
192.168.10.3"

while read ip
do
	ssh $ip "echo hello"
done <<<"$ips"
```

结果：只会输出一次hello
说明：由于使用了"<<<" 这种格式，while循环语句里如果使用了ssh命令，将自动重置输入流，导致后面的变量都无法获取。

解决方法1: 使用for循环代替while循环，可以解决此问题

```
for ip in $ips
do
	ssh $ip "echo hello"
done
```


解决方法2: 使用其他文件描述符号

```
while read -u10 ip
do
	ssh $ip "echo hello"
done 10<<<"$ips"
```
