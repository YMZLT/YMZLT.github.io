---
title: USTC计科复试刷题记录/2019
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:19:34
tags:
	- 刷题
	- 代码
categories:
	- 考研
---

# USTC计科复试刷题记录

[【参考】](https://zdszero.github.io/posts/ustc-test-2021/)

## 2019年

### 1. 身份证校验码 【数学】

一个合法的身份证号码前17个数字加1位校验码组成。校验码的计算规则如下：

首先对前17位数字加权求和，权重分配为：{7，9，10，5，8，4，2，1，6，3，7，9，10，5，8，4，2}；然后将计算的和对11取模得到值`Z`；最后按照以下关系对应`Z`值与校验码`M`的值：

```cpp
Z：0 1 2 3 4 5 6 7 8 9 10
M：1 0 X 9 8 7 6 5 4 3 2
```

现给定一个身份证号，从标准输入读取，判断该身份证是否合理，如果合理输出"YES"，如果不合理输出"NO"。

```cpp
输入示例:
37070419881216001X
输出示例:
NO
```

```cpp
#include <iostream>
using namespace std;

int w[17] = {7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2};
char m[11] = {'1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'};

bool isValid(string &s) {
  int sum = 0;
  for (int i = 0; i < 17; i++) {
    if (!isdigit(s[i])) {
      return false;
    }
    sum += w[i] * (s[i] - '0');
  }
  return (m[sum % 11] == s[17]);
}

int main() {
  string s;
  cin >> s;
  cout << (isValid(s) ? "YES" : "NO");
  return 0;
}
```

### 2. 二进制的意义【数学】

判断给定的数字是否可以拆分为两个2的幂的和的形式，如果可以输出拆分方案，不能输出"NO"。

```cpp
输入示例:
5
输出示例:
5 = 2^0 + 2^1

输入示例:
7
输出示例:
NO
```

```cpp
#include<cstdio>
#include<vector>

using namespace std;

int main()
{
    int num,temp;
    scanf("%d",&num);
    temp=num;
    vector<int> e; // 两个2的幂
    int i=0; // 记录二进制1在的位置
    while(temp!=0){
        if((temp&1)==1)
            e.push_back(i);
        temp=temp>>1;
        i++;
    }
    // 判断该数字对应二进制位是否有两个1即可。
    if(e.size()!=2) // 转换成二进制只能有2个1
        printf("NO\n");
    else{
        printf("%d = 2^%d + 2^%d\n",num,e[0],e[1]);
    }
    return 0;
}
```

### 3.前缀表达式求值 【栈】

给出前缀式，只有加减乘除，求结果。前缀式从文件`pre.in`中读取，将结果输出到标准输出中，题目保证输入的前缀式有效。

```cpp
输入示例:
- + 1 * + 2 3 4 5
输出示例:
16
```

```cpp
#include<fstream>
#include<iostream>
#include<stack>

using namespace std;
//- + 1 * + 2 3 4 5
int main()
{
    ifstream ifs("pre.in");
    stack<char> inputs; // 存放输入字符的栈
    stack<int> operaters; // 存放操作数的栈
    string s;
    getline(ifs, s); // 获取一行字符串
    for(char ch:s){
        if(ch==' ') continue;
        inputs.push(ch);
    }
    while(!inputs.empty()){
        char top=inputs.top();
        if(isdigit(top)){
            // 数字加入到操作数栈
            operaters.push(top-'0');
        }
        else{
            // 取操作数
            int l_op=operaters.top();
            operaters.pop();
            int r_op=operaters.top();
            operaters.pop();
            // 计算结果并存入操作数栈中
            if(top=='+')
                operaters.push(l_op+r_op);
            else if(top=='-')
                operaters.push(l_op-r_op);
            else if(top=='*')
                operaters.push(l_op*r_op);
            else if(top=='/')
                operaters.push(l_op/r_op);
        }
        inputs.pop();
    }
    cout<<operaters.top();
    return 0;
}
```

### 4. 集合划分 【回溯 】

求集合的所有划分，集合从文件`set.in`中读取，结果输出到标准输出中。

```cpp
输入示例:
a b c
输出示例:
{{a,b,c}}
{{a,b},{c}}
{{a,c},{b}}
{{a},{b,c}}
{{a},{b},{c}}
```

```cpp
#include<iostream>
#include<cstdio>
#include<vector>
using namespace std;
void solve(vector<int>& nums);
int main()
{
    int n;
    scanf("%d",&n);
    vector<int> nums(n);
    for(int i=0;i<n;i++)
        scanf("%d",&nums[i]);
    solve(nums);
    return 0;
}
void printVec(vector<vector<int> >&ans)
{
    if(ans.size()==0) return;
    printf("{");
    for(int i=0;i<ans.size();i++)
    {
        printf("{");
        for(int j=0;j<ans[i].size();j++)
        {
            if(j==ans[i].size()-1)
                printf("%d",ans[i][j]);
            else
                printf("%d,",ans[i][j]);
        }
        if(i==ans.size()-1)
            printf("}");
        else
            printf("},");
    }
    printf("}\n");
}
void subsets(vector<int>& nums,vector<vector<int>> &ans,int idx)
{
    if(idx==nums.size())
    {
        printVec(ans);
        return ;
    }
    // 新加入一个集合
    ans.push_back(vector<int>(1,nums[idx]));
    subsets(nums,ans,idx+1);
    ans.pop_back();

	// 加入当前所有集合
    for(int i=0;i<ans.size();i++)
    {
        ans[i].push_back(nums[idx]);
        subsets(nums,ans,idx+1);
        ans[i].pop_back(); // 回溯
    }

    return;
}
void solve(vector<int> &nums)
{
    vector<vector<int>> ans;
    subsets(nums,ans,0);
}
```

### 5. 最大路径和【二叉树】

给出一个**二叉排序树**的层次遍历，求从它的一个叶子结点到另一个叶子结点的路径，要求路径上经过结点的数值之和最大。二叉树的层次遍历序列从文件`expr.in`中读取，结点数值大于0，将结果输出到标准输出中。

```cpp
输入示例:
25 15 40 5 20 30 50 10 35
输出示例:
165
```

```cpp
#include<fstream>
#include<iostream>

using namespace std;

// 二叉树节点
struct node
{
    int val;
    node* lchild,* rchild;
    node(int x):val(x),lchild(NULL),rchild(NULL) {};
};
// 向二叉排序树中插入节点
void insertNode(node* &root, int val)
{
    // 空树直接创建节点并返回
    if(root==NULL)
    {
        root=new node(val);
        return ;
    }
    // 按照二叉排序树的规则找到符合插入要求的节点
    node* pre; //记录插入的根节点
    node* r=root;
    while(r)
    {
        pre = r;
        if(val<=r->val) r=r->lchild;
        else r=r->rchild;
    }
    if(val<=pre->val)
        pre->lchild=new node(val);
    else
        pre->rchild=new node(val);
}
// 计算二叉树根节点到叶子节点 路径值之和最大的
int maxPath(node* root)
{
    // 后序遍历
    if(root==NULL)
        return 0;
    int left=maxPath(root->lchild);
    int right=maxPath(root->rchild);
    return root->val+(left>right?left:right);
}

int main()
{
    ifstream ifs("input.in");

    int num;
    node *root=NULL; // 新建二叉树根节点
    // 读入数据并构造二叉排序树
    while (ifs >> num)
    {
        insertNode(root,num);
    };
    //一个子树内部的最大路径和 = 左子树提供的最大路径和 + 根节点值 + 右子树提供的最大路径和
    cout<<maxPath(root->lchild)+maxPath(root->rchild)+root->val;
    return 0;
}
```