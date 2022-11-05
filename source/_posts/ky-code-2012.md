---
title: USTC计科复试刷题记录/2012
top: false
cover: false
toc: true
mathjax: true
tags:
  - 刷题
  - 代码
categories:
  - 考研
abbrlink: a38fe35d
date: 2022-10-31 18:07:23
---


本文为2012年中科大计算机学院复试机试刷题记录，使用C++作为编程语言，并在Linux 平台下使用gcc编译器进行编译以及测试运行。输入输出风格偏向于C，有些题目会用到C++的STL标准函数库。

<!--more-->

## 题目详解

### 字符串处理

> 字符串处理：从`string.in`文件里读入两个字符串，字符串除了数字还可能包括`'-'、'E'、'e'、'.'`，相加之后输出到文件`string.out`中，如果是浮点型，要求用科学计数法表示（最多保留10个有效数字）。

```
输入示例:                       
34.56                                
2.45e2    

输出示例: 
2.7956e2 
```
- 思路：可以通过是否有小数点判断输入的是整数还是浮点数，有小数点则利用atof()将字符串转为double型，否则使用atoi()转换为int型。然后就可以用`"%.10e"`输出科学计数法表示的浮点数。但是这样还有一个问题，就是浮点数输出来结果会有多余的0和加号，`2.7956000000e+02`。可以先将结果字符串输出到缓冲区，然后消除多余的0以及加号等。

  具体代码如下：

```cpp
#include <cstdio>
#include <cstdlib>

int main()
{
    FILE *infile = fopen("string.in", "r");
    FILE *outfile = fopen("string.out", "w");
    char str1[128];
    char str2[128];
    fscanf(infile, "%s", str1);
    fscanf(infile, "%s", str2);

    int i = 0;
    bool flag_double_str1 = false, flag_double_str2 = false;
    while (str1[i] != '\0')
    {
        if (str1[i] == '.')
        {
            flag_double_str1 = true;
            break;
        }
        i++;
    }
    i = 0;
    while (str2[i] != '\0')
    {
        if (str2[i] == '.')
        {
            flag_double_str2 = true;
            break;
        }
        i++;
    }
    if (!flag_double_str1 && !flag_double_str2) // 全为整形则直接输出
    {
        int a = atoi(str1);
        int b = atoi(str2);
        fprintf(outfile, "%d", a + b);
        return 0;
    }
    // 若有一个数为浮点数则转为double
    double a = atof(str1);
    double b = atof(str2);
    char buf[128];
    sprintf(buf, "%.10e", a + b);
    int j;
    for (i = 0; buf[i] != '\0'; i++)
        if (buf[i] == 'e')
            break;
    for (j = i - 1; buf[j] == '0'; j--)
        ; // 从e往前找，跳过中间为0的部分
    for (int k = 0; k <= j; k++)
        fputc(buf[k], outfile);
    fputc('e', outfile);
    i++;
    while (buf[i] == '+' || buf[i] == '0') // 消除e后的加号以及0
        i++;
    for (j = i; buf[j] != '\0'; j++)
        fputc(buf[j], outfile);
    return 0;
}
```

- 测试

```shell
$cc -o ex1 ex1.cpp
// string.in
34.56                                
2.45e2 
// string.out
2.7956e2
// string.in
34
2
// string.out
36
```



### 最大公约数

> 最大公约数：从`number.in`文件中读入n个数，求出这n个数的最小值、最大值以及它们两的最大公约数，输出到文件`number.out`中。number.in 中第一行为n，接下来为n个大于零的整数。 

```
输入示例:
3
4 8 6

输出示例:
4 8 4
```

- 思路：首先遍历数组nums得到最大值与最小值后，再计算两者的最大公约数即可。求最大公约数有三种常用的方法：**辗转相除法**、**更相减损术** 及其基础上进行改进的 **Stein算法**。

- 辗转相除法，又称欧几里得算法，对于两个非负整数$x,y(x>y)$，其计算公式为：

$$
gcd(x,y)=gcd(y,x\ mod \ y)
$$

- 更相减损术，出自《九章算术》（原本是为了求约分，修改后可用来求解最大公约数）的一种求最大公约数的算法。主要思路为：以较大的数减较小的数，接着把所得的差与较小的数比较，并以大数减小数。继续上述操作，直到所得的减数和差相等为止。

- 参考力扣题目：[1979. 找出数组的最大公约数](https://leetcode.cn/problems/find-greatest-common-divisor-of-array/)

采用辗转相除法得到的代码如下：

```c++
#include <cstdio>

int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }

int main()
{
    FILE *in_fp;
    FILE *out_fp;

    in_fp = fopen("number.in", "r");
    out_fp = fopen("number.out", "w");

    int n;
    fscanf(in_fp, "%d", &n);

    int nums[n];
    for (int i = 0; i < n; i++)
    {
        fscanf(in_fp, "%d", &nums[i]);
    }
    int min, max;
    min = max = nums[0];
    for (int i = 1; i < n; i++)
    {
        if (nums[i] < min)
            min = nums[i];
        if (nums[i] > max)
            max = nums[i];
    }
    fprintf(out_fp, "%d %d %d", min, max, gcd(max, min));
    fclose(in_fp);
    fclose(out_fp);
    return 0;
}
```

- 测试

```shell
$cc -o ex2 ex2.cpp
$./ex2
//number.in
3
4 8 6
//number.out
4 8 4
//number.in
7
4 8 6 10 9 24 30
//number.out
4 30 2
```



### 任务调度

> 任务调度：从`task.in`文件中读入任务调度序列，输出n个任务适合的一种调度方式到`task.out`中。每行第一个表示前序任务，括号中的任务为后序任务，只有在前序任务完成的情况下，后序任务才能开始。若后序为NULL则表示无后序任务。

```
输入示例:
Task0(Task1,Task2)
Task1(Task3)
Task2(NULL)
Task3(NULL)

输出示例:
Task0 Task1 Task3 Task2
```

- 思路：这道题是典型的拓朴排序问题。拓扑排序可以使用深度优先遍历，在DFS实现拓扑排序时，用**栈**来保存拓扑排序的顶点序列；并且保证在某顶点入栈前，其所有邻接点已入栈。
- 练习题目：[P1347 排序](https://www.luogu.com.cn/problem/P1347)
- 一般来说树和图的输入都比较复杂，还有建树，建图这种操作。所以刷树和图的时候，我比较推荐用洛谷上的题目，这上面的题目都是有输入输出要求的，跟考试的要求差不多，而力扣是函数式的编程，无法锻炼处理输入输出的能力。

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <stack>
using namespace std;

#define N 10           // 记录最大顶点数
#define INF 0x7fffffff // 定义无穷大
int graph[N][N];       // 有向图邻接矩阵
int n = 0;             // 记录顶点个数
stack<int> S;

void DFS(int u, bool *visit)
{
    visit[u] = true;
    // 遍历找到所有没被访问过的邻接节点进行访问
    for (int v = 0; v < n; v++)
    {
        if (graph[u][v] == 1 && !visit[v])
        {
            DFS(v, visit);
        }
    }
    S.push(u);
}

int main()
{
    FILE *in_fp = fopen("task.in", "r");
    FILE *out_fp = fopen("task.out", "w");
    // 初始化图矩阵
    for (int i = 0; i < N; i++)
    {
        for (int j = 0; j < N; j++)
            graph[i][j] = INF;
    }
    // 读入有向图
    char buf[20];
    while (fscanf(in_fp, "%s", buf) != EOF)
    {
        int r;
        r = atoi(&buf[4]);
        graph[r][r] = 0;
        int i = 4;
        while (buf[i] != '\0')
        {
            if (buf[i] == 'k') // 转化Task后面的数组
            {
                int v = atoi(&buf[i + 1]);
                graph[r][v] = 1; // 记录有向图的一条边
            }
            i++;
        }
        n++;
    }
    bool visit[n] = {false}; // 记录顶点是否访问
    // 深度优先遍历记录拓扑排序 注意可能图不连通
    for (int i = 0; i < n; i++)
    {
        if (visit[i])
            continue;
        DFS(i, visit);
    }
    // 打印拓扑排序
    while (!S.empty())
    {
        fprintf(out_fp, "Task%d ", S.top());
        S.pop();
    }
    fclose(in_fp);
    fclose(out_fp);
    return 0;
}
```

- 测试

```shell
$g++ -o ex3 ex3.cpp
$./ex3
// task.out
Task0 Task2 Task1 Task3 
```

拓扑排序的结果可能有多种，这里使用邻接矩阵从最小的序号开始深度优先遍历，所以与题目给的样例不一样。

### 火车票订购

> 火车票订购：火车经过X站，火车的最大载客人数为m，有n个订票请求，请求订购从a站到b站的k张票，若能满足订购要求则输出1，否则输出0。数据从`ticket.in`中输入，第一行有两个数字，分别是n，m。接下来有n行，每行三个数分别是a、b、k。结果输出到文件`ticket.out`中。

```
输入示例:
5 10
4 10 9
8 12 2
8 12 1
14 20 8
30 300 15

输出示例:
1
0
1
1
0
```
- 思路：用一个大数组来存储每一站的车上人数，在每一次运送乘客时先尝试是否有足够空位，如果有空位则将flag置为1，否则将flag置为0，如果尝试之后flag=1则可以搭载这波乘客。

```cpp
#include<fstream>
#include<vector>

using namespace std;

ifstream ifs("input.in");
ofstream ofs("output.out");
int n,m; //火车的最大载客人数为m，有n个订票请求
vector<int> record(1000,0); // 每站路车上人数

int main()
{

    ifs>>n>>m;
    vector<int> ans(n);
    for(int i=0; i<n; i++)
    {
        int a,b,k; //请求订购从a站到b站的k张票
        ifs>>a>>b>>k;
        int j;
        for(j=a; j<b; j++) // 检查每个路段是否合法
        {
            if(record[j]+k>m)
            {
                ans[i]=0;
                break;
            }
        }
        if(j!=b) continue;
        // 检查合法则订购成功 记录本次订购
        ans[i]=1;
        for(int j=a; j<b; j++)
        {
            record[j]+=k;
        }
    }
    for(int i=0; i<(int)ans.size(); i++)
    {
        ofs<<ans[i]<<endl;
    }
}
```



### 最短路径

> 最短路径：有n个城市和m条道路（n < 1000, m < 10000），每条道路有不同的长度，请找到从起点s到终点t的最短距离，并且输出经过的城市的序号，如果有多条，输出字典序最小的那条；若从s到t没有路径，则输出“can't arrive”。从文件`road.in`中读入数据，第一行有四个数，分别为n、m、s、t。接下来有m行，每行三个数，分别为两个城市的序号和相互之间的距离，将结果输出结果到文件`road.out`中。

```
输入示例:
3 3 1 3
1 3 3
1 2 1
2 3 1

输出示例:
2
1 2 3
```

- 思路：这道题考察的是单源最短路径问题，有`Dijkstra`算法和多源最短路径的`floyed-warshall`算法，这里使用比较容易编程实现的`floyed-warshall`算法，三重循环搞定。用road二维数组记录图的信息，用next二维数组记录下一跳。
- 参考题目：[P4779 【模板】单源最短路径（标准版）](https://www.luogu.com.cn/problem/P4779)

```cpp
#include<fstream>
#include<vector>
#include<climits>
using namespace std;
const int INF=INT_MAX>>1;
ifstream ifs("input.in");
ofstream ofs("output.out");
int n,m,s,t;

int main()
{
    ifs>>n>>m>>s>>t;
    vector<vector<int> >g(n+1,vector<int>(n+1,INF)); // 无向图邻接矩阵
    vector<vector<int> >next(n+1,vector<int>(n+1,0)); // next[i][j]表示i到j最短路径的下一个顶点
    for(int i=0; i<m; i++)
    {
        int u,v,w;
        ifs>>u>>v>>w;
        g[u][v]=w;      // 无向图边初始化
        g[v][u]=w;
        next[u][v]=v;   // 将有边连接的顶点下一个顶点初始化
        next[v][u]=u;
    }
    for(int i=1; i<=n; i++)
    {
        g[i][i]=0;
        next[i][i]=i;
    }
    for(int x=1; x<=n; x++) // 弗洛伊德算法求多源最短路径
    {
        for(int i=1; i<=n; i++)
        {
            for(int j=1; j<=n; j++)
            {
                if(g[i][x]==INF||g[x][j]==INF) continue;
                int sum=g[i][x]+g[x][j];
                if(sum<g[i][j])
                {
                    g[i][j]=sum; // 松弛边
                    next[i][j]=x; // 记录最短路径的中间结点
                }
            }
        }
    }
    if(g[s][t]==INF)
    {
        ofs<<"can't arrive."<<endl;
        return 0;
    }
    ofs<<g[s][t]<<endl;
    ofs<<s;
    int cur=s;
    while(cur!=t){
        cur=next[cur][t];
        ofs<<" "<<cur;
    }
    return 0;
}
```

# 参考链接

- [2012年中科大考研上机机试试题（回忆版）](http://www.cskaoyan.com/thread-87090-1-1.html)
- [中科大2012年复试机试题详解](https://zdszero.github.io/posts/ustc-test-2012/)