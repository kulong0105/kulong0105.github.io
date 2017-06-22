---
title: GDB常用命令简介
category: linux
tags:
- linux
---

## 介绍

本文将对Linux下的代码调试工具GDB的常用命令进行介绍。

GDB是GNU开源组织发布的一个强大的Linux下的程序调试工具, 主要用于调试C/C++程序。


## 常用命令

### backtrace

显示栈信息

<!--more-->

### break

设置断点

- break \<function\>
- break memset (Glibc函数）
- break filename:linenum
- break filename:function
- break \*address (在程序运行的内存地址处停住）
- break ... if condition (break ... if i=100)
- break \<linenum\> thread \<threadID\>


### continue

恢复运行

- contiue


### catch

设置捕捉点

- catch \<event\>


### cd / pwd /make

- cd
切换目录

- pwd
显示当前目录

- make
执行Makefile文件


### clear/delete/disable/enable

维护停止点

- clear \[linenum\]      (如： clear 519）
- delete \[breakpoint\]  (如：delete 1-2)
- disable \[breakpoint\] (如：disable 3-10）
- disable display <dnums>
- enable  \[breakpoint\] (如: enable 3-10)
- enable display <dnums>


### condition

停止条件维护

- condition <bnum> <expression> (修改断点号为bnum的停止条件为expression)
- condition <bnum> (清除断点号为bnum的停止条件)
- condition [bnum] ... end (为停止点设定运行命令)

```
	break foo if x > 0
		commands
		printf "x is %d, x"
		continue
	end
```


### display

当程序停住时或单步跟踪时，会自动显示指定的表达式

- display \<expr\>
- display /\<f\> \<expr\>
- undisplay <dnums>

ex:
`display /i $pc` ( /i表示输出机器指令码，$pc是环境变量，表示当前指定地址)


### directory

- direcotry \<dirname : dirname\> 增加一个源文件路径
- directory 清楚所有的自定义的源文件搜索路径信息
- show directories 显示定义了的原文件搜索路径


### disassemable

- disassemable 查看当前函数的汇编代码
- disassemable <func_name> 查看指定函数的汇编代码


### examine

使用命令x来查看指定内存地址中的值

x /<n,f,u> <addr>

- n: 表示显示内存的长度，即显示几个数据，每个数据的长度由第三个参数指定
- f：表三显示的格式，如果地址所指的是字符串，可以用s，如果是指定地址，可以用i
- u：不指定表示4bytes，b-单字节，h-双字节，w-四字节，g-八字节

x/3uh  0x54320  以双字节为一个单位取值，取3个


### file

加载被调试的可执行文件

- file program

### gdb

调试已运行的程序

- gdb \<program\> PID
- gdb \<program\>; attch/deattach PID


### handle

信号处理

handle \<signal\> \<action\>

action:
- nostop/stop
- nopass/pass
- noprint/print


### info

- info line (查看源代码在内存中的地址）
- info program (查看程序是否在运行）
- info breakpoints (显示所有断点）
- info watchpoints (显示所有观察点）
- info frame (显示当前栈帧信息）
- info args (显示当前函数的局部变量）
- info locals (显示当前函数的局部变量）
- info threads (查看线程列表）
- info handle （查看信号列表）
- info registers (查看寄存器信息）
- info all-registers (查看所有寄存器信息，包括浮点寄存器）
- info source (查看源代码信息）
- info display （查看display设置信息）


### jump

跳转执行

- jump \<linenum\> / \<filename:linenum\>
- jump \<address\> (代码行的内存地址）

jump命令不会改变当前的程序栈中的内容，所以从一个函数跳到另一个函数时，
当函数运行完进行弹栈操作时，必然会发生错误。

jump命令是通过修改eip寄存器的值来实现，于是，可以手工进行设置：set $pc=0x4850


### list

显示源代码

- linenum
- filename:linenum
- filename:function


### path dir

增加设定程序的运行路径


### print

* 查看运行时数据 print /\<format\> \<expr\>

* 查看一段连续的内存空间的值

int \*a = malloc(sizeof(int)\*2);
int b[2];

print \*a@2 (2表示长度）
print b     (b是数组，可直接使用）

* 当全局变量和局部变量重名时，查看全局变量需要使用"::"操作符
	* p file::variable
	* p function::variable

* 修改变量值  print x=4   把变量x的值修改为4


### return

强制函数返回

- return
- return \<expression\> (该表达式的值会被作为函数返回值）

如果调试断点在某个函数中，并还有语句没有执行完，使用return命令可以强制函数忽略
还没有执行的语句并返回


### run

开始运行程序


### search/reverse-search

搜索源代码

- serach <regexp> (向前搜索）
- reverse-search <regexp> (向后搜索）


### shell

执行外部命令

- shell ls
- shell df


### set

- args: 指定运行时参数

- env var name=val:设置环境变量

- listsize <count>: 设置listsize

- print <pretty / addr / array / ... >  <on / off>: 设置print设置

- language val: 设置调试语言

- `$var=val`
可以定义变量用来保存调试程序中的运行数据（没有类型，可以定义任一类型), `set $i=0; print bar[$i++]`

- scheudler-locking off:
多线程时，gdb会根据实际的运行情况不断地切换当前跟踪的线程，会使得调试困难，
不过可以使用如下设置固定当前调试的线程（慎用，可能会引起死锁）

- follow-fork-mode child / detach-on-fork on:
这两个命令设定后，fork时将挂起父进程，进入子进程。
如把child改为parent，fork时挂起子进程，继续调试父进程。


### show

- args: 查看设置好的运行参数
- paths:  查看程序设定的运行路径
- env: 显示环境变量
- listsize: 显示listsize
- language:
- print <pretty / addr / array / ... >


### signal

发送信号

signal \<signal\> (信号直接发送给被调试程序）


### step/stepi/next/nexti

单步调试

- step: 执行一行源程序代码，如果此行代码中有函数调用，则进入该函数；
- next: 执行一行源程序代码，此行代码中的函数调用也一并执行。
- stepi: 类似step，针对汇编
- nexti: 类似next，针对汇编


### util

退出当前循环体


### watch

设置观察点

- watch \<expr\>  (表达式expr发生变化，程序停住）
- rwatch \<expr\> (表达式expr被读时，程序停住）
- awatch \<expr\> (表达式的值被读或写时，程序停住）


### whatis

查看类型

- whatis name
