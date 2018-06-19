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

### typeset使用

* typeset 用在函数内部，如果不使用-g选项，相当于local定义功能
* typeset -a  定义的数组是通过下标ID进行索引的， typeset -A 可以根据内容进行索引
```
typeset  -a  student
student[0]="AAA"
student[1]="BBB"

typeset -a teacher
teacher["math"]="CCC"
teacher["english"]="DDD"
```

### exec 使用

符号	意义
n>&m	将FD为m的输出复制到FD为n的文件
n<&m	将FD为m的输入复制到FD为n的文件
n>&-	关闭FD为n的输出
n<&-	关闭FD为n的输入
&>file	将stdout和stderr重定向到file
[j]<>file  把文件"file"打开, 并且将文件描述符"j"分配给它,该文件描述符可用于读写

more details can refer to
1) http://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/x13082.html
2) http://molinux.blog.51cto.com/2536040/469554


### 检查远程端口是否打开

除了可以使用nmap命令检测远程端口是否打开，还可以通过/dev/tcp/$ip/$port 文件来判断

```
$ echo > /dev/tcp/$ip/$port &>/dev/null  &&  echo "Open"
```


### 数组操作

定义： 
a=(1 2 3 4 5)

获取长度：
echo ${#a[*]}

读取：
echo ${#a[2]}

删除：
unset a[1]

切片:
echo ${a[@]:2}


### here文档

- 对变量进行反斜杠转义，避免变量替换

```
cat >/tmp/testfile <<-EOF
abc
\$edf
EOF
```

- 不需要对变量进行反斜杠转义，可以直接避免变量替换

```
cat >/tmp/testfile <<-"EOF"
abc
\$edf
EOF
```

- 合并文件

```
cat - /tmp/file1 > /tmp/file2 <<EOF
abc
123
EOF
```


### awk 移除重复的行

```
awk '!x[$0]++' /tmp/testfile
```

refers: https://unix.stackexchange.com/questions/159695/how-does-awk-a0-work
