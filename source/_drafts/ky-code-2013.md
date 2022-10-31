---
title: USTC计科复试刷题记录/2013
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:15:04
tags:
	- 刷题
	- 代码
categories:
	- 考研
---



[【参考】](https://zdszero.github.io/posts/ustc-test-2013/)

## 2013年

### 4.分解质数

在`4.in`中有一个正整数，把这个正整数分解成两个质数的和的方案数输出到`4.out`.

```
输入示例:
10
输出示例:
2 （5+5=10  3+7=10   7+3和3+7是一个方案）
```

```cpp
#include<cstdio>
#include<cmath>
#include<vector>
using namespace std;
bool isPrime(int n) // 判断n是否为质数
{
    if((n==1)||((n&1)==0)) return false;
    for(int i=3;i<=(int)sqrt(n);i++)
    {
        if(n%i==0) // 如果n有除了它本身和1以外的因子 则不是质数
            return false;
    }
    return true;
}
int main()
{
    int n;
    scanf("%d",&n);
    vector<pair<int,int> > ans;
    for(int i=1;i<=(n)>>1;i++){
        if(isPrime(i)&&isPrime(n-i))
            ans.push_back(make_pair(i,n-i));
    }
    int cnt=ans.size();
    printf("%d (",cnt);
    for(int i=0;i<cnt;i++)
    {
        printf("%d+%d=%d",ans[i].first,ans[i].second,n);
        if(i!=cnt-1) printf(" ");
    }
    printf(")\n");
    return 0;
}
```



### 5.N皇后

n皇后问题：n×n格的棋盘上放置彼此不受攻击的n个皇后。按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。n后问题等价于在n×n的棋盘上放置n个皇后，任何2个皇后不妨在同一行或同一列或同一斜线上。在文件`5.in`中给定棋盘的大小n，在文件`5.out`中输出放置方法的个数。

```
输入示例:
4
输出示例:
2
```

### 6.字符串匹配

字符串匹配问题：在文件`6.in`中有两个字符串，在文件`6.out`输出第二个字符串在第一个字符串中的起始和终止位置，如果没有则输出0。

```
输入示例:
abcdefgdrefege
cdef
输出示例:
3 6
```

- `kmp`算法

### 7.哈夫曼树

哈夫曼树问题：在文件`7.in`中有几个数，和为1，进行哈夫曼编码，并把编码结果输出到`7.out`中。

```
输入示例:
0.1 0.15 0.2 0.25 0.3
输出示例:
000
001
01
10
11
```

```cpp
#include <fstream>
#include <queue>
#include <unordered_map>
#include <vector>
using namespace std;

struct node
{
    node *lchild, *rchild;
    double val;
    node(double val) : lchild(nullptr), rchild(nullptr), val(val) {}
};
ifstream ifs("input.in");
ofstream ofs("output.out");
unordered_map<node *, int> idxMap;
vector<string> ans;
string tmp;

void preOrder(node *root)  // 前序遍历得到哈夫曼编码
{
    if (root==nullptr)
        return;
    if (root->lchild==nullptr && root->rchild==nullptr) // 遍历到叶子结点则停止 将当前的编码保存下来
    {
        ans[idxMap[root]] = tmp;
        return;
    }
    tmp += "0";
    preOrder(root->lchild);
    tmp.pop_back();

    tmp += "1";
    preOrder(root->rchild);
    tmp.pop_back();
}

struct Cmp
{
    bool operator()(node *l, node *r)
    {
        return l->val > r->val;
    }
};

int main()
{
    priority_queue<node *, vector<node *>, Cmp> pq; // 小顶堆 保存结点
    int idx = 0;
    double num;
    while (ifs >> num)
    {
        node *t = new node(num);
        pq.push(t);
        idxMap[t] = idx++;
    }
    ans.resize(pq.size());
    while (pq.size() != 1)  // 构造哈夫曼树
    {
        node *t1 = pq.top();
        pq.pop();
        node *t2 = pq.top();
        pq.pop();
        node *t = new node(t1->val + t2->val);
        t->lchild = t1;
        t->rchild = t2;
        pq.push(t);
    }
    node *root = pq.top(); // 哈夫曼树根节点

    preOrder(root);
    for (string &s : ans)
    {
        ofs <<s << endl;
    }
    return 0;
}
```

