---
title: C语言的tpyedef简介
category: C
tags:
- typedef
---

## 介绍

本文将详细介绍C语言下typedef的各种用法。


### 1. typedef创建易于记忆的类型名
```c
typedef int size;
```

说明:
此声明定义了一个 int 的同义字，名字为 size。注意 typedef 并不创建新的类型。


### 2. typedef掩饰数组类型
```c
char line[81];
char text[81];
typedef char allen[81];
allen text, line;
```

<!--more-->

### 3. typedef掩饰指针类型
```c
typedef char * pstr;
```

标准函数 strcmp()有两个const char *类型的参数。因此，它可能会误导人们像下面这样声明 strcmp()

```c
int strcmp(const pstr, const pstr); 
```

说明：
这是错误的，按照顺序，const pstr被解释为char * const，而不是const char *。这个问题很容易解决：
```c
typedef const char * cpstr;
int strcmp(cpstr, cpstr); // 现在是正确的
```

注：只要为指针声明typedef，那么都要在typedef名称中加一个const.


### 4. typedef代码简化
```c
typedef int (*PF) (const char *, const char *)；　
```
说明：
这个声明引入了 PF 类型作为函数指针的同义字，该函数有两个 const char * 类型的参数以及一个 int 类型的返回值。如果要使用下列形式的函数声明，那么上述这个 typedef 是不可或缺的：

```c
PF Register(PF pf);
```

说明：
Register() 的参数是一个 PF 类型的回调函数，返回某个函数的地址，其署名与先前注册的名字相同。做一次深呼吸。下面我展示一下如果不用 typedef，我们是如何实现这个声明的：
```c
int (*Register (int (*pf)(const char *, const char *)))(const char *, const char *); 
```

### 5. typedef  & 存储类关键字
typedef 就像 auto，extern，mutable，static， register一样，是一个存储类关键字，表明typedef 会真正影响对象的存储特性。 
```c
typedef register int FAST_COUNTER;   // 错误，编译通不过。
```

说明：
问题出在不能在声明中有多个存储类关键字。因为符号 typedef 已经占据了存储类关键字的位置，在 typedef 声明中不能用 register（或任何其它存储类关键字）。


### 6. typedef促进跨平台开发
typedef 有另外一个重要的用途，那就是定义机器无关的类型。例如，你可以定义一个叫 REAL 的浮点类型，在目标机器上它可以获得最高的精度：
```c
typedef long double REAL;
```

说明：
1）在不支持 long double 的机器上，该 typedef 看起来会是下面这样：typedef double REAL; 
2）在连 double 都不支持的机器上，该 typedef 看起来会是这样：typedef float REAL; 


### 7. typedef  &  结构体
当用下面的代码定义一个结构时，编译器报了一个错误，莫非C语言不允许在结构中包含指向它自己的指针吗？
```c
typedef struct tagNode
{
	char *pItem;
	pNode pNext;
} *pNode;
```

说明：
原因在于新结构建立的过程中遇到了pNext域的声明，类型是pNode，要知道pNode表示的是类型的新名字，那么在类型本身还没有建立完成的时候，这个类型的新名字也还不存在，也就是说这个时候编译器根本不认识pNode。

正确使用如下：
```c
typedef struct tagMyStruct
{ 
int iNum;
long lLength;
} MyStruct; 
```
说明：
这语句实际上完成两个操作：
* 定义一个新的结构类型
```c
struct tagMyStruct
{ 
	int iNum; 
	long lLength; 
}; 
```

* typedef为这个新的结构起了一个名字，叫MyStruct。
```c
typedef struct tagMyStruct MyStruct; 
```

tagMyStruct实际上是一个临时名字，struct 关键字和tagMyStruct一起，构成了这个结构类型，不论是否有typedef，这个结构都存在。
我们可以用struct tagMyStruct varName来定义变量，但要注意，使用tagMyStruct varName来定义变量是不对的，因为struct 和tagMyStruct合在一起才能表示一个结构类型。

如下做法也正确：
```c
typedef struct tagNode 
{
	char *pItem;
	struct tagNode *pNext;
} *pNode; 

typedef struct tagNode *pNode;
struct tagNode 
{
	char *pItem;
	pNode pNext;
};
```
 
注意：在这个例子中，你用typedef给一个还未完全声明的类型起新名字。C语言编译器支持这种做法。

### 8. typedef  &  #define的问题:
有下面两种定义pStr数据类型的方法，两者有什么不同，哪一种更好一点？
```c
typedef char *pStr;
#define pStr char *; 
```

通常讲，typedef要比#define要好，特别是在有指针的场合。请看例子：
```c
typedef char *pStr1;
#define pStr2 char *;
pStr1 s1, s2;
pStr2 s3, s4; 
```

说明：
在上述的变量定义中，s1、s2、s3都被定义为char*，而s4则定义成了char，不是我们所预期的指针变量，根本原因就在于#define只是简单的字符串替换而typedef则是为一个类型起新名字。

总结#define与typedef使用：
* #define宏定义有一个特别的长处：可以使用 #ifdef ,#ifndef等来进行逻辑判断，还可以使用#undef来取消定义。
* typedef也有一个特别的长处：它符合范围规则，使用typedef定义的变量类型其作用范围限制在所定义的函数或者文件内（取决于此变量定义的位置），而宏定义则没有这种特性。

### 9. typedef & 复杂的变量声明
下面是三个变量的声明，使用typdef分别给它们定义一个别名
```c
int *(*a[5])(int, char*);
void (*b[10]) (void (*)());
doube(*)() (*pa)[9];   //这个太狗血了~
```

说明：
对复杂变量建立一个类型别名的方法很简单，你只要在传统的变量声明表达式里用类型名替代变量名，然后把关键字typedef加在该语句的开头就行了。 

* example1: 
```c
int *(*a[5])(int, char*);   
typedef int *(*pFun)(int, char*); //pFun是我们建的一个类型别名
pFun a[5];  //使用定义的新类型来声明对象，等价于int* (*a[5])(int, char*);
```

* example2: 
```c
void (*b[10]) (void (*)());
typedef void (*pFunParam)();//首先为上面表达式蓝色部分声明一个新类型
typedef void (*pFun)(pFunParam);//整体声明一个新类型
pFun b[10];//使用定义的新类型来声明对象，等价于void (*b[10]) (void (*)());
```

* example3: 
```c
doube(*)() (*pa)[9]; 
typedef double(*pFun)();//首先为上面表达式蓝色部分声明一个新类型
typedef pFun (*pFunParam)[9];//整体声明一个新类型
pFunParam pa; //使用定义的新类型来声明对象，等价于doube(*)() (*pa)[9];

