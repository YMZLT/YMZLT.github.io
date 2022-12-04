---
title: 中科大计算机系统作业4
top: false
cover: false
toc: true
mathjax: false
tags:
  - 作业
  - CSAPP
  - 链接
  - fork
categories:
  - 作业
abbrlink: fbb78c73
date: 2022-12-04 16:21:55
---

本文为中科大研究生课程——计算机系统作业4的题目解答记录。使用的教材为《深入理解计算机系统》（第3版）。

<!--more-->

# 题目与解答

## 7.7

不改变任何变量名字，修改 7. 6. 1 节中的 bar5.c, 使得 foo5.c 输出 x 和 y 的正确值（也就是整数
15213 和 15212 的十六进制表示）。

7.6.1：

```c
/* foo5.c */
#include <stdio.h>
void f(void);

int y = 15212;
int X = 15213;

int main()
{
	f();
	printf("x = 0x%x y = 0x%x \n",x, y);
	return O;
}
/* bar5.c*/
double x;
void f()
{
    x = -0.0;
}
```

**解：**将bar5.c中的x声明为静态变量，使其链接为内部链接，只在当前文件中起作用

```c
/* bar5.c (修改后)*/
static double x; // 将bar5.c中的x声明为静态变量，使其链接为内部链接，只在当前文件中起作用
void f()
{
    x = -0.0;
}
```



## 7.8

在此题中， REF(x.i)---->DEF(x.k) 表示链接器将任意对模块 i 中符号 x 的引用与模块 k 中符号 x 的定义相关联。在下面每个例子中，用这种符号来说明链接器是如何解析在每个模块中有多重定义的引用的。如果出现链接时错误（规则 1) 写"错误（ERROR）"。如果链接器从定义中任意选择一个(规则 3) 那么写"未知（unkown）"。

**解：**

A. 链接器选择定义在模块 1 中的强符号，而不是定义在模块 2 中的弱符号（规则 2)   

(a) REF(main.1) -> DEF(main.1)

(b) REF(main.2) -> DEF(main.2)

B. 模块1，2中的 x 均为弱符号，如果有多个弱符号同名，那么从这些弱符号中任意选择一个（规则 3) 。

(a) REF(x.1) -> DEF(unkown)

(b) REF(x.2) -> DEF(unkown)

C. 这是一个错误，因为每个模块都定义了一个强符号 x ( 规则 1) 。  

(a) REF(x.1) -> DEF(ERROR)

(b) REF(x.2) -> DEF(ERROR)

## 7.10

a 和 b 表示当前路径中的目标模块或静态库，而 a--->b 表示 a 依赖于 b, 也就是说 a 引用了一个 b 定义的符号。对于下面的每个场景，给出使得静态链接器能够解析所有符号引用的最小的命令行（即含有最少数量的目标文件和库参数的命令）。

A.  p.o--->libx.a--->p.o

B.  p.o--->libx.a--->liby.a 和 liby.a--->libx.a

C.  p.o--->libx.a--->liby.a--->libz.a 和 liby.a--->linx.a--->libz.a

**解：**

A.

```shell
gcc p.o libx.a
```

B.

```shell
gcc p.o libx.a liby.a libx.a
```

C.

```shell
gcc p.o libx.a liby.a libx.a libz.a 
```



## 7.12

考虑目标文件 m.o 中对函数 swap 的调用（作业题 7. 6) 。

```assembly
9: e8 00 00 00 00		callq e <main+0xe> swap()
```

具有如下重定位条目：

```assembly
r.offset = 0xa
r.symbol = swap
r.type = R_X86_64_PC32
r.addend = -4
```

A. 假设链接器将 m.o 中的 .text 重定位到地址 0x4004e0, 把 swap 重定位到地址 0x4004f8 。那么callq 指令中对 swap 的重定位引用的值应该是什么？

B. 假设链接器将 m.o 中的 .text 重定位到地址 0x4004d0, 把 swap 重定位到地址 0x400500 。那么callq 指令中对 swap 的重定位引用的值应该是什么？  

**解：**

A.

```assembly
ADDR(s) = ADDR(.text) = 0x4004e0
ADDR(r.symbol) = ADDR(swap) = 0x4004f8 
```

链接器首先计算引用的运行时地址：  

```assembly
refaddr = ADDR(s) + r.offset
		= 0x4004e0 + 0xa
		= 0x4004ea
```

然后修改此引用：  

```assembly
*refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr)
		= (unsigned) (0x4004f8 + (-4) - 0x4004ea)
		= (unsigned) (0xa)
```

因此，得到的可执行目标文件中，对 swap 的 PC 相对引用的值为 0xa

B.

```assembly
ADDR(s) = ADDR(.text) = 0x4004d0
ADDR(r.symbol) = ADDR(swap) = 0x400500 
```

链接器首先计算引用的运行时地址：  

```assembly
refaddr = ADDR(s) + r.offset
		= 0x4004d0 + 0xa
		= 0x4004da
```

然后修改此引用：  

```assembly
*refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr)
		= (unsigned) (0x400500 + (-4) - 0x4004da)
		= (unsigned) (0x22)
```

因此，得到的可执行目标文件中，对 swap 的 PC 相对引用的值为 0x22

## 8.13

下面程序的一种可能的输出是什么？  

```c
/* code/ecflforkprob3.c */
#include "csapp.h"
int main()
{
    int x = 3;
    if(Fork()!=0) // Fork() 是 fork() 函数的错误处理包装函数
        printf("x=%d\n",++x);
    printf("x=%d\n",--x);
    exit(0);
}
```

**解：**

- fork 函数只被调用一次，却会返回两次：一次是在调用进程（父进程）中，一次是在新创建的子进程中。在父进程中， fork 返回子进程的 PID 。在子进程中，fork  返回 0 。父进程和子进程是并发运行的独立进程，都有自己的私有地址空间。
- 当 fork 调用在第 6 行返回时，在父进程和子进程中 x 的值都为 3，子进程在第 7 行加一并输出它的 x 的副本。相似地，父进程在第 8 行减一并输出它的 x 的副本。  

所以最终可能输出：（保证4在3前面）

```c
# case1:
x=4; // 父进程执行第7行printf语句
x=3; // 父进程执行第8行printf语句
x=2; // 子进程执行第8行printf语句

# case2:
x=4; // 父进程执行第7行printf语句
x=2; // 子进程执行第8行printf语句
x=3; // 父进程执行第8行printf语句

# case3:
x=2; // 子进程执行第8行printf语句
x=4; // 父进程执行第7行printf语句
x=3; // 父进程执行第8行printf语句
```



## 8.15

下面这个程序会输出多少个 "hello" 输出行？  

```c
/* code/ecf/forkprob6.c */
#include "csapp.h"
void doit()
{
    if(Fork()==0){
        Fork();
        printf("hello\n");
        return;
    }
    return;
}
int main()
{
    doit();
    printf("hello\n");
    exit(0);
}
```

**解：**

![](csapp-hw4/hw4.svg)

所以共输出5个"hello"行

## 8.18

考虑下面的程序：  

```c
/* code/ecflforkprob2.c */
#include "csapp.h"
void end(void)
{
    printf("2");fflush(stdout);
}
int main()
{
    if(Fork()==0)
        atexit(end);
    if(Fork()==0){
        printf("0");fflush(stdout);
    }
    else{
        printf("1");fflush(stdout);
    }
    exit(0);
}
```

判断下面哪个输出是可能的。注意： atexit 函数以一个指向函数的指针为输入，并将它添加到函数列表中（初始为空），当 exit 函数被调用时，会调用该列表中的函数。  

A. 112002	B. 211020 	C. 102120	D. 122001   E. 100212  

**解：**

![](csapp-hw4/hw4-2.svg)

构成拓扑排序的只有ACE正确。

B中的第一个不可能是2。D中的第一个1后面不可能有两个2。

