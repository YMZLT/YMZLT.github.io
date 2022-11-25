---
title: 中科大计算机系统作业2
top: false
cover: false
toc: true
mathjax: true
tags:
	- 作业
	- CSAPP
categories:
	- 作业
abbrlink: 12d42946
date: 2022-10-30 17:19:05s
---

本文为中科大研究生课程——计算机系统作业2的题目解答记录。使用的教材为《深入理解计算机系统》（第3版）。

<!--more-->

# 题目与解答

## 3.58  

一个函数的原型为

```c
long decode2(long x, long y, long z);
```

GCC 产生如下汇编代码：

```assembly
decode2:
	subq 	%rdx, %rsi
	imulq 	%rsi, %rdi
	movq 	%rsi, %rax
	salq 	$63,%rax
	sarq 	$63, %rax
	xorq 	%rdi, %rax
	ret
```

参数x 、y 和z 通过寄存器%rdi、%rsi 和 %rdx 传递。代码将返回值存放在寄存器%rax 中。写出等价于上述汇编代码的decode2 的C代码。

**解：**

```c
long decode2(long x, long y, long z)
{
  long tmp = y - z;
  return (tmp * x) ^ (tmp << 63 >> 63);
}
```

## 3.60  

考虑下面的汇编代码：

```c
long loop(long x, int n)
x in%rdi, n in%esi
```

```assembly
loop :
	movl	%esi, %ecx
	movl	$1, %edx
	movl	$0, %eax
	jmp		.L2
.L3:
	movq	%rdi, %r8
	andq	%rdx, %r8
	orq		%r8, %rax
	salq	%cl, %rdx
.L2:
	testq %rdx, %rdx
	jne .L3
	rep; ret
```

以上代码是编译以下整体形式的C 代码产生的：

```c
long loop2(long x, int n)
{
    long result = ____________;
    long mask;
    for (mask = ____________; mask ____________; mask =____________){
        result |= ____________;
    }
    return result;
}
```

你的任务是填写这个C 代码中缺失的部分，得到一个程序等价于产生的汇编代码。回想一下，这个函数的结果是在寄存器%rax 中返回的。你会发现以下工作很有帮助：检查循环之前、之中和之后的汇编代码，形成一个寄存器和程序变最之间一致的映射。

A. 哪个寄存器保存着程序值x 、n 、result 和mask?

B. result 和mask 的初始值是什么？

C. mask 的测试条件是什么？

D. mask 是如何被修改的？

E. result 是如何被修改的？

F. 填写这段C 代码中所有缺失的部分。

**解：**

A. 

| 变量   | 寄存器 |
| :----- | :----- |
| x      | %rdi   |
| n      | %esi   |
| result | %rax   |
| mask   | %rdx   |

B. 

```c
result = 0
mask = 1
```

C. 

```c
mask != 0
```

D. 

```
mask = mask << n
```

E. 

```c
long loop2(long x, int n) {
  long result = 0;
  long mask;
  for (mask = 1; mask != 0; mask <<= n) {
    result |= (x & mask);
  }
  return result;
}
```

## 3.63  

这个程序给你一个机会，从反汇编机器代码逆向工程一个switch 语句。在下面这个过程中，去掉了switch 语句的主体：

```c
long switch_prob(long x, long n)
{
    long result = x;
    switch (n){
    /* Fill in code here */
            
    }
    return result;
}
```

下面给出了这个过程的反汇编机器代码：

```c
long switch_prob (long x,long n)
x in%rdi , n in%rsi
```

```assembly
0000000000400590 <switch_prob>:
400590: 48 83 ee 3c				sub		$0x3c,%rsi
400594: 48 83 fe 05				cmp		$0x5,%rsi
400598: 77 29					ja		4005c3 <switch_prob+0x33>
40059a: ff 24 f5 f8 06 40 00	jmpq	*0x4006f8(,%rsi,8)
4005a1: 48 8d 04 fd 00 00 00	lea		0x0 (,%rdi,8),%rax
4005a8: 00
4005a9: c3						retq	
4005aa: 48 89 f8				mov		%rdi,%rax
4005ad: 48 c1 f8 03				sar		$0x3,%rax
4005b1: c3						retq	
4005b2: 48 89 f8				mov		%rdi,%rax
4005b5: 48 c1 eO 04				shl		$0x4,%rax
4005b9: 48 29 f8				sub		%rdi,%rax
4005bc: 48 89 c7				mov		%rax,%rdi
4005bf: 48 Of af ff				imul	%rdi,%rdi
4005c3: 48 8d 47 4b				lea		Ox4b(%rdi),%rax
4005c7: c3						retq	
```

跳转表驻留在内存的不同区域中。可以从第5 行的间接跳转看出来，跳转表的起始地址为0x4006f8 。用调试器GDB, 我们可以用命令x/6gx 0x4006f8 来检查组成跳转表的6个8 字节字的内存。GDB 打印出下面的内容：

```assembly
(gdb) x/6gx 0x4006f8
0x4006f8: 0x00000000004005a1 	0x00000000004005c3
0x400708: 0x00000000004005a1 	0x00000000004005aa
0x400718: 0x00000000004005b2	0x00000000004005bf
```

用C 代码填写开关语句的主体，使它的行为与机器代码一致。

**解：**

```c
long switch_prob(long x, long n)
{
    long result = x;
    switch (n){
        case 60:
        case 62:
            result = 8*x;
            break;
        case 63:
            result = x>>3;
            break;
        case 64:
            result= (x<<4)-x;
            x=result;
        case 65:
            x=x*x;
        default:
            result = 0x4B+x;  
    }
    return result;
}
```

## 3.69  

你负责维护一个大型的C 程序，遇到下面的代码：

```c
typedef struct
{
    int first;
    a_struct a[CNT];
    int last;
} b_struct;

void test(long i, b_struct *bp)
{
    int n = bp->first + bp->last;
    a_struct *ap = &bp->a[i];
    ap->x[ap->idx] = n;
}
```

编译时常数CNT 和结构a_struct 的声明是在一个你没有访问权限的文件中。幸好，你有代码的'.o'版本，可以用OBJDUMP 程序来反汇编这些文件，得到下面的反汇编代码：

```c
void test (long i, b_struct *bp)
i in %rdi, bp in %rsi
```

```assembly
0000000000000000 <test>:
	0: Sb Se 20 01 00 00	mov		0x120(%rsi),%ecx
	6: 03 Oe				add 	(%rsi),%ecx
	S: 4S Sd 04 bf			lea 	(%rdi,%rdi,4),%rax
	c: 4S Sd 04 c6			lea 	(%rsi,%rax,8),%rax
	10: 4S Sb 50 OS			mov 	Ox8(%rax),%rdx
	14: 4S 63 c9			movslq	%ecx,%rcx
	17: 48 89 4c dO 10		mov		%rcx,0x10(%rax,%rdx,8)
	le: c3					retq
```

运用你的逆向工程技术，推断出下列内容：

A. CNT 的值。

B. 结构a_struct 的完整声明。假设这个结构中只有字段idx 和x , 并且这两个字段保存的都是有符号值。

**解：**

A. 7

```assembly
0000000000000000 <test>:
	0: Sb Se 20 01 00 00	mov		0x120(%rsi),%ecx  # Get bp->last
	6: 03 Oe				add 	(%rsi),%ecx # n = bp->first
```

int 类型4字节，还需要进行对齐，所以占8字节，剩下的都是数组`a_struct a[CNT];`的空间，
`CNT*40 + 8 = 288 = 0x120`，所以`CNT = 7`。

B. 

分析汇编代码：

```assembly
0000000000000000 <test>:
	0: Sb Se 20 01 00 00	mov		0x120(%rsi),%ecx
	6: 03 Oe				add 	(%rsi),%ecx # get n
	S: 4S Sd 04 bf			lea 	(%rdi,%rdi,4),%rax  # compute 5*i
	c: 4S Sd 04 c6			lea 	(%rsi,%rax,8),%rax  # compute bp+40*i
	10: 4S Sb 50 OS			mov 	Ox8(%rax),%rdx # Get bp->a[i].idx
	14: 4S 63 c9			movslq	%ecx,%rcx # shift n from 32b to 64b
	17: 48 89 4c dO 10		mov		%rcx,0x10(%rax,%rdx,8) # mov n to ap->x[ap->idx] 
	le: c3					retq
```

第6行得到idx的值，这里使用的%rdx寄存器，所以应该是long类型；由第一小问可以知道`a_struct `的大小为40字节，第7行将n从32字节转为64，所以数组x应该是long类型，并且为4个。所以`a_struct `的完整声明如下：

```c
typedef struct {
  long idx;
  long x[4];
} a_struct;
```

## 3.70

考虑下面的联合声明：

```c
union ele{
    struct{
        long *p;
        long y;
    } e1;
    struct{
        long x;
        union ele *next;
    } e2;
};
```

这个声明说明联合中可以嵌套结构。下面的函数（省略了一些表达式）对一个链表进行操作，链表是以上述联合作为元素的：

```c
void proc(union ele *up){
    up->___________= *(___________) - ___________;
}
```

A. 下列字段的偏移址是多少（以字节为单位）：

```c
e1.p    _____________
e1.y	_____________
e2.x	_____________
e2.next	_____________
```

B. 这个结构总共需要多少个字节？

C. 编译器为proc 产生下面的汇编代码：

```c
void proc (union ele *up)
up in %rdi
```

```assembly
proc:
	movq	8(%rdi), %rax
	movq	(%rax) , %rdx
	movq	(%rdx), %rdx
	subq	8(%rax), %rdx
	movq	%rdx, (%rd)
	ret
```

在这些信息的基础上，填写proc 代码中缺失的表达式。提示：有些联合引用的解释可以有歧义，当你清楚引用指引到哪里的时候，就能够澄清这些歧义。只有一个答案，不需要进行强制类型转换，且不违反任何类型限制。

**解：**

A. 

```c
e1.p    0
e1.y	8
e2.x	0
e2.next	8
```

B. 共需要8字节。

C. 

```c
void proc(union ele *up){
    up->e2.x= *((up->e2.next)->e1.p) - (up->e2.next)->e1.y;
}
```

# 参考

[CSAPP-3E-SOLUTIONS]( https://dreamanddead.github.io/CSAPP-3e-Solutions/)