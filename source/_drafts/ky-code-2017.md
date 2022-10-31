---
title: USTC计科复试刷题记录/2017
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:19:26
tags:
	- 刷题
	- 代码
categories:
	- 考研
---



# USTC计科复试刷题记录

[【参考】](https://zdszero.github.io/posts/ustc-test-2021/)

## 2017年

### 1. 字符串操作【基础】

文本文件`input1.txt`中含有一行字符串（字符串长度不大于1000），由若干个英文单词构成，单词之间以一个或多个空格分割，请将该字符串中单词之间多余的空格删除，即单词之间只保留一个空格，字符串首尾的空格也删除，并将所有单词的首字母改成大写之后输出到屏幕。

```cpp
输入
i  am  very nervous today      since i   will attend the C   programming
输出
I Am Very Nervous Today Since I Will Attend The C Programming
```

```cpp
#include<iostream>
#include<fstream>
#include<vector>

using namespace std;
int main()
{
    ifstream ifs("input1.txt");
    string s;
    vector<string> words;
    while(ifs>>s){
        if(s[0]<='z'&&s[0]>='a')
            s[0]=s[0]-'a'+'A';
        words.push_back(s);
    }
    for(auto word:words) cout<<word<<" ";
    return 0;
}
```

### 2. 盈数和完数【数学 基础】

一个数如果等于它所有的因子（除了自己本身）之和，比如`6=3+2+1`，则称其为“完数”；若因子之和大于该数，则称其为“盈数”。在屏幕上输出2到1000之间的所有“盈书”和“完数”。

```cpp
#include<iostream>
#include<cmath>
using namespace std;

int solve(int n)
{
    int sum=1;
    for(int i=2;i<=n/2;i++)  // 除1以外 最小的因子>=2 最大的因子<=n/2
    	if(n%i==0)	sum+=i;
    return sum;
}
int main()
{
    for(int i=2;i<=1000;i++)
    {
        int num=solve(i);
        if(num==i)	cout<<i<<":完数"<<endl;
        else if(num>i)	cout<<i<<":盈数"<<endl;
    }
    return 0;
}
```

### 3. 最长等差子序列长度【DP】

文本文件`input3.txt`中有一个有序的正整数序列，以空格作为间隔符。计算该序列中包含的最长等差子序列的长度，并输出，例如

```cpp
输入：1 2 3 5 7 8 14 20 26
最长等差子序列：2 8 14 20 26
输出：5
```

- 动态规划

```cpp
#include <iostream>
#include <fstream>
#include <vector>
#include <map>
using namespace std;

int main(int argc, char *argv[])
{
    ifstream ifs("./input3.txt");
    int num;
    vector<int> v(1, 0);
    while (ifs >> num)
        v.push_back(num);
    int n = v.size() - 1;
    int maxv = 0;
    map<int, vector<int>> dp;
    for (int i = 1; i <= n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            int diff = v[i] - v[j];
            if (!dp.count(diff))
                dp[diff] = vector<int>(n+1, 1);
            int tmp = dp[diff][j] + 1;
            dp[diff][i] = tmp;
            if (tmp > maxv)
                maxv = tmp;
        }
    }
    cout << maxv << endl;
    return 0;
}

```



### 4. 割点【图】

在文本文件`input4.txt`中给定无向连通图G，判断图G是否存在这样一个顶点V，当V被删除时，该图的其他部分不再连通。如果存在，只需要找出一个这样的顶点，并输出顶点的编号， 如果不存在，则输出"not exist"。

```
输入：
// 表示顶点个数
// 节点1和5之间有一条边
5
1 5 
2 3
2 4
3 5
4 5
输入数据对应的图如下所示：
       4
      / \
1 -- 5   2
      \ /
       3
输出：
5
```

```cpp
#include <fstream>
#include <iostream>
#include <vector>
using namespace std;

// 按照深度优先遍历 从一个顶点遍历图
void dfs(vector<vector<int> > &g,int idx,vector<bool> &visited)
{
    for(int i=1; i<g.size(); i++)
    {
        if(g[idx][i]==1&&!visited[i]) // idx-i 相邻且未被访问过则继续遍历
        {
            visited[i]=true;
            dfs(g,i,visited);
        }
    }
}

int main(int argc, char *argv[])
{
    ifstream ifs("input4.txt");
    int n;
    ifs >> n; // 顶点个数
    // 用矩阵表示无向图 -1表示顶点之间无边 1 表示有边(顶点序号从1开始)
    vector<vector<int> > g(n+1,vector<int>(n+1,-1)); 
    int a, b;
    while (ifs >> a >> b) // 读入边 表示到矩阵中
    {
        g[a][b] = g[b][a] = 1; // 矩阵表示无向图是对阵矩阵
    }
    // 尝试删除一个顶点 判断是否连通
    for(int key=1; key<=n; key++)
    {
        // 删除所有跟key有关的边
        for(int i=1; i<=n; i++)
        {
            // 将原来有边的变为删除状态
            if(g[i][key]==1)
            {
                g[i][key]=0;
                g[key][i]=0;
            }
        }
        /*
        从任意一个节点开始深度优先遍历 用一个数组记录下遍历过的节点
        如果最后有未被遍历到的节点 则该图不联通
        */
        vector<bool> visited(n+1,false);
        visited[key==1?2:1]=true;
        dfs(g,key==1?2:1,visited); // 默认从1节点访问，如果删除节点为1 则从2节点开始访问
        for(int i=1; i<=n; i++)
        {
            if(i==key) continue; // 不考虑已经被删除的点
            if(visited[i]==false)
            {
                cout << key;
                return 0;
            }
        }
        // 剩下的图仍然连通 恢复被删除的节点 继续
        for(int i=1; i<=n; i++)
        {
            if(g[i][key]==0)
            {
                g[i][key]=1;
                g[key][i]=1;
            }
        }
    }
    cout << "not exist";
    return 0;
}
```



-  [Tarjan算法：求解图的割点与桥（割边）](https://www.cnblogs.com/nullzx/p/7968110.html)

```cpp
#include<fstream>
#include<cstdio>
#include<vector>
using namespace std;
int n; // 顶点数
ifstream ifs("input.in");
int dnf[100]= {0};
int low[100]= {0};
bool visited[100]; // 记录结点是否被访问
int deep=0;
void tarjan(vector<vector<int> >&g,int u,int f) // u记录当前访问的顶点  f记录当前访问的顶点的父节点
{
    deep++;
    visited[u]=true; // 标记该结点已经被访问
    dnf[u]=deep; // 更新该节点的访问次序
    low[u]=deep;
    for(int v=1; v<=n; v++)
    {
        if(g[u][v]==0||v==f) continue; //跳过无边连接的顶点 以及父结点
        if(visited[v]==false)   tarjan(g,v,u);
        if(low[u]>low[v])   low[u]=low[v]; // 用孩子结点的low 更新
    }
}

int main()
{
    ifs>>n;
    vector<vector<int> >g(n+1,vector<int>(n+1,0));
    int a,b;
    while(ifs>>a>>b)
    {
        g[a][b]=1;
        g[b][a]=1;
    }
    // 确定开始的根节点
    int r=1;
    tarjan(g,r,0);
    for(int i=1; i<=n; i++)
    {
        printf("%d %d\n",dnf[i],low[i]);
    }
    for(int u=2; u<=n; u++) // 排除根节点
    {
        for(int v=1; v<=n; v++)
        {
            if(g[u][v]==0) continue;
            if(low[v] >= dnf[u])
            {
                // 存在至少一个孩子顶点V满足low[v] >= dnf[u]，
                //就说明顶点V访问顶点U的祖先顶点，必须通过顶点U，
                //而不存在顶点V到顶点U祖先顶点的其它路径，所以顶点U就是一个割点。
                printf("%d",u);
                return 0;
            }
        }
    }
    // 判断根节点是否为割点 如果回溯到根顶点后，还有未访问过的顶点，需要在邻接顶点上再次进行DFS，根顶点就是割点。
    for(int i=1;i<=n;i++)
    {
        if(visited[i]==false) printf("%d",r);
    }
    return 0;
}
```

