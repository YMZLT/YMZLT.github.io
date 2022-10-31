---
title: USTC计科复试刷题记录/2016
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:19:22
tags:
	- 刷题
	- 代码
categories:
	- 考研
---

# USTC计科复试刷题记录

[【参考】](https://zdszero.github.io/posts/ustc-test-2021/)

## 2016

### 1. 进制转换【数学】

设计程序，将用户输入的十进制整数转换为十六进制数，并在屏幕上输出。

```cpp
#include<iostream>
#include<vector>
using namespace std;
int main()
{
    int num;
    cin>>num;
    vector<char> ans;
    while(num!=0)
    {
        int m=num%16;
        if(m<10)	ans.push_back(m+'0');
        else	ans.push_back(char('A'+m-10)); 
        num/=16;
    }
    for(int i=ans.size()-1;i>=0;i--)
        cout<<ans[i];
    return 0;
}
```



### 2. 微信红包分配【随机数 map】

编程实现一个简单的我微信红包分配算法。满足以下要求即可：

1. 键盘输入红包金额X元，以及红包个数N。X是正数，例如10.88，表示10.88元。N是正整数。
2. 每个红包的金额至少是一分钱。如果总金额X无法满足这一条件，则提示重新输入。
3. 在红包金额固定的条件下，单个红包的金额随机生成。
4. 在屏幕上输出红包分配结果。

```cpp
输入示例
100 5
输出示例
10.1  20  30.2  9.9  29.8
```

```cpp
#include<iostream>
#include<time.h>
#include<map>
using namespace std;

int main()
{
    double num;
    int total,n;
    do
    {
        cout<<"please input X and N:";
        cin>>num>>n;
        total=num*100;
    }
    while(total<n);  // 金额不够分则提示重新输入

    srand((unsigned) time(NULL));
    map<int,int> positions; // 记录切割位置
    int i=1;
    positions.emplace(0,1); // 头部位置
    while(i<n)	// 在[1,total-1] 上 取n-1个数字 即为切割位置
    {
        int pos;
        do{
            pos=rand()%(total-1)+1;
        }while(positions.find(pos)->second==1);  //如果该位置已经被切过了 则重新找一个切割位置
        positions.emplace(pos,1);
        i++;
    }
    positions.emplace(total,1); // 尾部位置

    auto prev = positions.begin();
    for (auto it = next(positions.begin()); it != positions.end(); it++)
    {
        printf("%.2f ", (it->first - prev->first) / 100.0);
        prev++;
    }
    return 0;
}
//please input X and N:1 10
//0.08 0.09 0.14 0.10 0.01 0.06 0.15 0.21 0.09 0.07
```



### 3. 多项式的根【数学 分治法】

求多项式的根$2x^{11} - 3x^8 - 5x^3 - 1 = 0$，根的精度为0.00000001

> 解题思路：二分法
>
> f(1)<0,f(2)>0  初始结果范围(1, 2)

```cpp
#include<iostream>
#include<cmath>
using namespace std;
double fun(double x)
{
    return 2*pow(x,11)-3*pow(x,8)-5*pow(x,3)-1;
}
int main()
{
    double l=1,r=2;
    while(l<r)
    {
        double mid=l+(r-l)/2.0;
        double res=fun(mid);
        if(fabs(res)<1e-9){
            printf("%.9lf\n",mid);
            return 0;
        }
        else if(res>0)	r=mid;
        else	l=mid;
    }
    return 0;
}
```

```cpp
#include <iostream>
#include <cmath>
using namespace std;

double func(double x)
{
    return 2 * pow(x, 11) - 3 * pow(x, 8) - 5 * pow(x, 3) - 1;
}
// 递归法
double solve(double (*func) (double), double l, double r)
{
    if (r - l < 0.00000001)
        return l;
    double mid = (r + l) / 2;
    double tmp = func(mid);
    if (tmp > 0) return solve(func, l, mid);
    else if (tmp < 0) return solve(func, mid, r);
    else return mid;
}

int main(int argc, char *argv[])
{
    printf("%.9f\n", solve(func, 1, 2));
    return 0;
}
```



### 4. 中序遍历【二叉树】

给定一个二叉树的前序遍历和后序遍历，给出一种可能的中序遍历结果。 

输入从文件`input.in`中给定。其中第一行是二叉树结点的个数，第二行是二叉树的前序遍历序列，第三行是后序遍历序列。二叉树中的结点名称以大写字母表示，最多26个结点。 将结果输出到文件`output.out`，输出一种可能的中序遍历结果。

```
输入示例
5
A B D C E
D B E C A
输出示例
D B A E C
```

```cpp
#include<iostream>
#include<fstream>
#include<vector>
using namespace std;
struct node
{
    char val;
    node* lchild;
    node* rchild;
    node(char ch):val(ch),lchild(NULL),rchild(NULL){} // 定义构造函数
};
node* createTree(vector<char>& pre,vector<char>& post,int preL,int preR,int postL,int postR)
{
    if((preL>preR) || (postL>postR)) return NULL;
    node* root= new node(pre[preL]);
    if(preL==preR) return root;

    // 左子树的根节点为pre [preL+1]，因此只要找到它在后序中的位置就可以分开左右子树
    int k;
    for(k=postL; k<=postR; k++)
    {
        if(post[k]==pre[preL+1])
            break;
    }
    int numLeft=k-postL+1; // 左子树结点个数
    // 前序区间划分:[preL][preL+1,preL+numLeft][preL+numLeft+1,preR]
    // 后序区间划分:[postL,k][k+1,postR-1][postR]

    // 生成左子树的根节点 作为根节点的左孩子
    root->lchild=createTree(pre,post,preL+1,preL+numLeft,postL,k);
    // 生成右子树的根节点 作为根节点的右孩子
    root->rchild=createTree(pre,post,preL+numLeft+1,preR,k+1,postR-1);
    return root;
}
ofstream ofs("output.out");
void inOrder(node* root)
{
    if(root==NULL) return ;
    inOrder(root->lchild);
    printf("%c ",root->val); // 访问根节点
    ofs<<root->val<<" ";
    inOrder(root->rchild);
}
int main()
{
    ifstream ifs("input.in");
    int n;
    ifs>>n;
    vector<char> pre(n),post(n);
    for(int i=0; i<n; i++)
    	ifs>>pre[i];
    for(int i=0; i<n; i++)
    	ifs>>post[i];
    node* root=createTree(pre,post,0,n-1,0,n-1); // 构建二叉树
    inOrder(root);
    return 0;
}
```

