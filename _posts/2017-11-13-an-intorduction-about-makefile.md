---
title: Makefile简介
category: linux
tags:
- linux
---

## 介绍

一个项目通常有多个源文件组成，关于如何编译这些源文件，Makefile定义了一套规则决定:
- 哪些文件要先编译
- 哪些文件后编译
- 哪些文件要重新编译

整个编译操作，只要运行一条`make`命令即可。

本文将对makefile的基本规则/语法/使用进行介绍，并进行一些实践操作。

<!--more-->

## Makefile

### 基本规则

```
目标: 依赖
[tab] 规则
```

说明：
-  "目标"是必需的，不可省略
-  "目标"可以是多个，以空格分隔
-  "依赖"和"规则"是可选的，但两者之中必须至少存在一个
-  第二行必须由一个tab键开始


### 工作原理

目标是否存在:
- 存在: 
    - 检查目标的所有依赖，任何一个依赖有更新时，就重新生成目标
    - 目标文件比依赖文件时间晚，则需要更新
- 不存在: 
    - 检查规则中的依赖文件是否存在
    - 不存在的话寻找是否有规则用来生成该依赖文件

<div style="text-align: center">
<img src="https://github.com/kulong0105/kulong0105.github.io/raw/master/documents/pictures/makefile.webp"/>
</div>


### 变量

- 内置变量
    - .RECIPEPREFIX 
    ```
    .RECIPEPREFIX = >   // 指定使用">"代替默认的tab键
    ```

    - .PHONY
    ```
    .PHONY: clean  // 明确声明clean是"伪目标", 不管是否存在clean文件，都执行clean目标所对应的规则
    ```

    - .EXPORT_ALL_VARIABLES
    ```
    .EXPORT_ALL_VARIABLES:   // 明确声明导出之后所有的变量到子进程
    ```

    - MAKECMDGOALS
    ```
    $(info *** starting Makefile for goal(s) "$(MAKECMDGOALS)")  //打印编译目标
    ```

    - SHELL
    ```
    SHELL := /bin/bash   // 设置shell
    ```

    - MAKEFLAGS
    ```
    MAKEFLAGS += -s    // 关闭执行命令时回显
    MAKEFLAGS += --no-builtin-rules  // 关闭内建规则
    ```

- 自动变量
    - $@
    ```
    abc efg: test
    ```
    `$@`表示当前目标，如上述规则中，`$@`就表示abc和efg

    - $<
    ```
    test: a b c
    ```
    `$<`表示第一个前置条件，如上述规则中，$<就表示a
    
    - $?
    ```
    test: a b c
    ```
    `$?`表示比目标更新的所有前置条件，之间以空格分隔。如上述规则中, b的时间戳比test新，`$?`就表示b
    
    - $^
    ```
    test: a b c
    ```
    `$^`表示所有前置条件，如上述规则中，`$^`就表示a b c
    
    - `$*`
    ```
    %.o: %.c
    ```
    `$*`表示%匹配的部分, 比如%匹配main.c中的main, `$*`就表示main
    

- 环境变量

    - 读取OS的HOME环境变量
    ```
all:
    echo $${HOME}
    ```

- 普通变量
    - 执行时扩展
    ```
    v1 = $(v2)
    ```
    
    - 定义时扩展
    ```
    v1 := $(v2)
    ```
    
    - 空值才设置
    ```
    v1 ?= $(v2)
    ```
    
    - 变量追加
    ```
    v1 += $(v2)  // v1 = v1 + " " + v2
    ```

    - define变量
    ```
define HELP_INFO
#Usage:
#    make $target
#Example:
#    make tool
endef
help:
    echo $$HELP_INFO     // 使用$$引用变量
    ```


### 判断和循环

- 判断
```
ifeq ($(NAME),kulong)
    name="allen"
else
    name"lucky"
endif
```

- 循环
```
LIST = one two three
all:
    for i in $(LIST); do \
        echo $$i; \
    done
```


### 函数

- 内置函数
    - subst
    ```
    $(subst from,to,text)
    $(subst he,HE,hello)  //HEllo
    ``` 
    
    - wildcard
    ```
    files := $(wildcard /tmp/test/*.sh)
    ```
    
    - shell
    ```
    pwd : = $(shell pwd)
    ```
    
    - patsubst
    ```
    $(patsubst pattern,replacement,text)
    $(patsubst %.c,%.o,bar.c)  //bar.o
    ```
    
    - =.
    ```
    bak: $(OUTPUT:.sh=.sh.bak)  // 将变量OUTPUT中的后缀名 .sh 全部替换成 .sh.bak
    ```

    - notdir
    ```
    $(notdir names...)  
    $(notdir $(wildcard /tmp/test/*)) //获取文件名
    ```


    - filter
    ```
    $(filter pattern...,text)
    $(filter abc, $(wildcard /tmp/*/)) // 选择包含abc pattarn的文件
    ```

    - filter-out
    ```
    $(filter-out pattern...,text)
    $(filter-out abc, $(wildcard /tmp/*/)) // 选择不包含abc pattarn的文件
    ```
    - foreach 
    ```
    $(foreach var,list,text)
    dirs := a b c d
    files := $(foreach dir,$(dirs),$(wildcard $(dir)/*)) // 多个输出采用空格分隔
    ```
   
    - call
    ```
    $(call variable,param,param,...)
    reverse = $(2) $(1)
    foo = $(call reverse,a,b)  // foo 的结果为 b a 
    ```

    - error
    ```
    $(error  "fatal error")  //退出执行
    ```

    - warning
    ```
    $(warning "warning info") //告警，不退出
    ```

    - info
    ```
    $(info "tips: xxx") //提示信息
    ```


### 特殊符号

```
all:
    - sdf
    @ # some comments
    @ echo test
```

- -: 表示此命令即使执行出错，也依然继续执行后续命令
- @: 表示该命令只执行，不回显


## make

make命令的常用选项：

|    选项     |    描述   |
|    ---      |    ---    |
|-B| |不管当前目标是否是最新的，总是执行目标的规则|
|-C| |切换到指定目录去执行make命令|
|-d| |debug模式|
|-e| |设置环境变量|
|-f| |指定使用make的文件|
|-i| |忽略生成目标时执行规则遇到的错误|
|-I| |指定搜索Makefile文件的路径，当Makefile文件中使用include命令时有用|
|-j| |指定多少个job同时运行|
|-n| |打印将要执行的命令，但不执行|
|-p| |打印makefiles数据库|
|-s| |不打印出执行的命令|


## 错误使用

- 错误用法1:
```
[renyl@localhost build]$ cat -A Makefile
name = allen$
$
target:$
^I@echo "X"$
$
^Iifeq ($(name), allen)$         <=== 使用tab开头
^I^I@echo "Y"$
^Iendif$
$
[renyl@localhost build]$ make
X
ifeq (allen, allen)
/bin/sh: -c: line 0: syntax error near unexpected token `allen,'
/bin/sh: -c: line 0: `ifeq (allen, allen)'
make: *** [Makefile:5: target] Error 1
[renyl@localhost build]$
```

- 正确用法1:
```
[renyl@localhost build]$ cat -A Makefile
name = allen$
$
target:$
^I@echo "X"$
$
ifeq ($(name), allen)$          <=== 没有使用tab开头
^I@echo "Y"$
endif$
$
[renyl@localhost build]$ make
X
Y
[renyl@localhost build]$
```

- 错误用法2：
```
[renyl@localhost build]$ cat -A Makefile
ifeq ($(MAKECMDGOALS),)$
^I$(error This Makefile requires an explicit rule to be specified)$          <=== 使用tab开头
endif$
$
.PHONY: test$
test$
^I@echo test$
[renyl@localhost build]$ make
Makefile:2: *** recipe commences before first target.  Stop.
[renyl@localhost build]$
```

- 正确用法2：
```
[renyl@localhost build]$ cat -A Makefile
ifeq ($(MAKECMDGOALS),)$
    $(error This Makefile requires an explicit rule to be specified)$        <=== 没有使用tab开头
endif$
$
.PHONY: test$
test$
^I@echo test$
[renyl@localhost build]$ make
Makefile:2: *** This Makefile requires an explicit rule to be specified.  Stop.
[renyl@localhost build]$
```

说明：
- 当ifeq紧接在target后面，不需要使用tab开头，同时ifeq里面需要使用tab开头
- 当ifeq不作为target的执行内容时，ifeq里面不需要使用tab开头


## 实例

```
BOOK_DIR     := /test/book
SOURCE_DIR   := text
OUTPUT_DIR   := out
EXAMPLES_DIR := examples

QUIET            = @

SHELL            =  bash
AWK              := awk
CP               := cp
EGREP            := egrep
HTML_VIEWER      := cygstart
KILL             := /bin/kill
M4               := m4
MV               := mv
PDF_VIEWER       := cygstart
RM               := rm -f
MKDIR            := mkdir -p
LNDIR            := lndir
SED              := sed
SORT             := sort
TOUCH            := touch

RELEASE_TAR   := mpwm-$(shell date +%F).tar.gz
RELEASE_FILES := README Makefile *.pdf bin examples out text

%.pdf: %.fo
    java -Xmx128M $(FOP) $(FOP_FLAGS) $< $@ $(FOP_OUTPUT)

.PHONY: release
release: $(BOOK_PDF_OUT)
    ln -sf $(BOOK_PDF_OUT) .
    tar --create                            \
        --gzip                              \
        --file=$(RELEASE_TAR)               \
        --exclude=CVS                       \
        --exclude=semantic.cache            \
        --exclude=*~                        \
        $(RELEASE_FILES)
    ls -l $(RELEASE_TAR)
```


## 参考

- [Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)
- [Makefile 快速参考](https://ftp.gnu.org/old-gnu/Manuals/make-3.79.1/html_chapter/make_15.html)
