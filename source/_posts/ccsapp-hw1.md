---
title: 中科大计算机系统作业1
top: false
cover: false
toc: true
mathjax: true
tags:
	- 作业
	- CSAPP
categories:
	- 作业
abbrlink: 8bdd78fc
date: 2022-10-30 14:50:28
---

本文为中科大研究生课程——计算机系统作业1的题目解答记录。使用的教材为《深入理解计算机系统》。

<!--more-->

# 题目与解答

## 2.58 
编写过程is_little_endian, 当在小端法机器上编译和运行时返回1, 在大端法机器上编译运行时则返回0 。这个程序应该可以运行在任何机器上，无论机器的字长是多少。

**解：**

```c
#include <stdio.h>

typedef unsigned char *byte_pointer; // 定义一个字节指针

int is_little_endian()
{
    int num = 0xff;                          // 存储在内存中为 0x00 0x00 0x00 0xff
    byte_pointer start = (byte_pointer)&num; // 强制转换为四个字节数组

    if (start[0] == 0xff) // 小端模式读取的第一个字节为0xff
        return 1;
    return 0; // 大端模式读取的第一个字节为0x00
}

int main()
{
    printf("%d\n", is_little_endian());
    return 0;
}
```

## 2.61

写一个C 表达式，在下列描述的条件下产生1, 而在其他情况下得到0 。假设x是int类型。

A. x的任何位都等于1 。

B. x的任何位都等于0 。

C. x的最低有效字节中的位都等于1 。

D. x的最高有效字节中的位都等于0 。

代码应该遵循位级整数编码规则，另外还有一个限制，你不能使用相等（==）和不相等（!=）测试。

**解：**

A. 

```c
!~x
```

B. 

```c
!x
```

C.  

```c
!((~x)&0xff)  或
!~(x|~0xff)
```

D. 

```c
!((x >> ((sizeof(int)-1) << 3)) & 0xff)
```

## 2.77  

假设我们有一个任务：生成一段代码，将整数变量x 乘以不同的常数因子K 。为了提高效率，我们想只使用 + 、- 和<<运算。对于下列K的值，写出执行乘法运算的C表达式，每个表达式中最多使用3个运算。

A. K=17

B. K=-7

C. K=60

D. K=-112

**解：**

A.

```c
(x<<4)+x
```

B.

```c
x-(x<<3)
```

C.

```c
(x<<6)-(x<<2)
```

D.

```c
(x<<4)-(x<<7)
```

## 2.84  

填写下列程序的返回值，这个程序测试它的第一个参数是否小于或者等于第二个参数。假定函数f2u 返回一个无符号32 位数字，其位表示与它的浮点参数相同。你可以假设两个参数都不是NaN 。两种0, +0 和-0 被认为是相等的。

```c
int float_le(float x, float y)
{
    unsigned ux = f2u(x);
    unsigned uy = f2u(y);

    /*Get the sign bits*/ 
    unsigned sx = ux >> 31;
    unsigned sy = uy >> 31;
    /*Give an expression using only ux, uy, sx, and sy*/
    return   ;
}
```

**解：**

```c
unsigned f2u(float x)
{
    return *(unsigned *)&x;
}
/*
float_le：测试它的第一个参数是否小于或者等于第二个参数 x<=y 则 true 
*/
int float_le(float x, float y) 
{
    //将浮点数按照无符号数解释（位不变）
    unsigned ux = f2u(x);
    unsigned uy = f2u(y);

    // 获得符号位 0或1
    unsigned sx = ux >> 31; 
    unsigned sy = uy >> 31;

    // 四种x<=y的情况返回true
    return (ux << 1 == 0 && uy << 1 == 0) || // 1. +0或者-0都为0(左移一位去除符号位对0影响)
           (sx && !sy) ||                    // 2. x为负（符号位sx为1），y为正（符号位sy为0）
           (!sx && !sy && ux <= uy) ||       // 3. x,y都为正时，x<=y等价于ux<=uy
           (sx && sy && ux >= uy);           // 4. x,y都为负时，x<=y等价于ux>=uy
}
```

## 2.89  

我们在一个int 类型为32 位补码表示的机器上运行程序。float类型的值使用32 位IEEE 格式，而double 类型的值使用64 位IEEE 格式。我们产生随机整数x 、y 和z, 并且把它们转换成double 类型的值：

```c
/*Create some arbitrary values*/
int x = random();
int y = random();
int z = random();
/*Convert to double*/
double dx = (double)x;
double dy = (double)y;
double dz = (double)z;
```

对于下列的每个C 表达式，你要指出表达式是否总是为1 。如果它总是为1, 描述其中的数学原理。否则，列举出使它为0的参数的例子。请注意，不能使用IA32 机器运行GCC 来测试你的答案，因为对于float 和double, 它使用的都是80 位的扩展精度表示。

```c
A. (float)x==(float)dx
B. dx-dy==(double)(x-y)
C. (dx+dy)+dz==dx+(dy+dz)
D. (dx*dy)*dz==dx*(dy*dz)
E. dx/dx==dz/dz
```

**解：**

A. 真。int类型转成double类型不会损失精度，此时，x 和 dx 转成float类型虽然可能会损失精度，但是在舍入原则相同的情况下得到的结果相同。

B. 假。当x-y得到的结果溢出时，转成double类型的结果与真实结果不同，如y=INT_MIN时。

C. 真。两个int类型转成double类型后，相加得到的结果仍然在double类型可表示范围内，且不会因为舍入丢失精度，所以是可以结合的。

D. 假。两个int类型转成double类型后，相乘得到的结果不一定在double类型可表示范围内，可能会发生溢出或者由于舍入导致失去精度，所以不具有结合性。

E. 假。当dx != 0 且 dz == 0时，左边结果为1，右边结果为无穷大，不相等。

## 2.91

大约公元前250 年，希腊数学家阿基米德证明了$\frac{223}{71}<\pi<\frac{22}{7}$。如果当时有一台计算机和标准库<math.h>, 他就能够确定$\pi$的单精度浮点近似值的十六进制表示为 0x40490FDB 。当然，所有的这些都只是近似值，因为$\pi$不是有理数。

A. 这个浮点值表示的二进制小数是多少？

B. $\frac{22}{7}$的二进制小数表示是什么？提示：参见家庭作业2. 83 。

C. 这两个$\pi$的近似值从哪一位（相对于二进制小数点）开始不同的？

**解：**

A.

```c
十六进制：
    0x40490FDB
二进制：
    0100 0000 0100 1001 0000 1111 1101 1011
浮点格式：
    0 10000000 10010010000111111011011    
    s:0 e=2^7=128 E=128-127=1 f=0.10010010000111111011011 M=1.10010010000111111011011
二进制小数：
    11.0010010000111111011011
```

B. $\frac{22}{7}=3+\frac{1}{7}$

```c
Y=1，2^k-1=7,k=3,y=001
二进制小数：11.001001001...(y=001)
```

C. 从小数点后第9位开始不同。

# 参考

[CSAPP-3E-SOLUTIONS]( https://dreamanddead.github.io/CSAPP-3e-Solutions/)

