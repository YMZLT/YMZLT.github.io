---
title: USTC计科复试刷题记录/2012
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:07:23
tags:
	- 刷题
	- 代码
categories:
	- 考研
---

[【参考】](https://zdszero.github.io/posts/ustc-test-2012/)

## 2012年

### 1.字符串处理

字符串处理：从`string.in`文件里读入两个字符串，字符串除了数字还可能包括`'-'、'E'、'e'、'.'`，相加之后输出到文件`string.out`中，如果是浮点型，要求用科学计数法表示（最多保留10个有效数字）。

```
输入示例:                       
34.56                                
2.45e2    

输出示例: 
2.7956e2 
```

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  double a, b;
  char buf[128];
  FILE *infile = fopen("./string.in", "r");
  FILE *outfile = fopen("./string.out", "w");
  if (!infile || !outfile) exit(1);
  fscanf(infile, "%lf", &a);
  fscanf(infile, "%lf", &b);
  sprintf(buf, "%.10e", a+b);
  int i, j;
  for (i = 0; buf[i] != '\0'; i++)
    if (buf[i] == 'e')
      break;
  for (j = i-1; buf[j] == '0'; j--); // 跳过中间为0的部分
  for (int k = 0; k <= j; k++)
    fputc(buf[k], outfile);
  fputc('e', outfile);
  for (j = i+1; buf[j] != '\0'; j++)
    fputc(buf[j], outfile);
  return 0;
}
```



### 2. 最大公约数

最大公约数：从`number.in`文件中读入n个数，求出这n个数的最小值、最大值以及它们两的最大公约数，输出到文件`number.out`中。number.in 中第一行为n，接下来为n个大于零的整数。 

```
输入示例:
3
4 8 6

输出示例:
4 8 4
```

- 辗转相除法

### 3.任务调度

任务调度：从`task.in`文件中读入任务调度序列，输出n个任务适合的一种调度方式到`task.out`中。每行第一个表示前序任务，括号中的任务为后序任务，只有在前序任务完成的情况下，后序任务才能开始。若后序为NULL则表示无后序任务。

```
输入示例:
Task0(Task1,Task2)
Task1(Task3)
Task2(NULL)
Task3(NULL)

输出示例:
Task0 Task1 Task3 Task2
```

- 拓朴排序

```cpp

```



### 4.火车票订购

火车票订购：火车经过X站，火车的最大载客人数为m，有n个订票请求，请求订购从a站到b站的k张票，若能满足订购要求则输出1，否则输出0。数据从`ticket.in`中输入，第一行有两个数字，分别是n，m。接下来有n行，每行三个数分别是a、b、k。结果输出到文件`ticket.out`中。

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



### 5.最短路径

最短路径：有n个城市和m条道路（n < 1000, m < 10000），每条道路有不同的长度，请找到从起点s到终点t的最短距离，并且输出经过的城市的序号，如果有多条，输出字典序最小的那条；若从s到t没有路径，则输出“can't arrive”。从文件`road.in`中读入数据，第一行有四个数，分别为n、m、s、t。接下来有m行，每行三个数，分别为两个城市的序号和相互之间的距离，将结果输出结果到文件`road.out`中。

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

- 使用`floyed-warshall`算法，用road二维数组记录图的信息，用next二维数组记录下一跳。

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

