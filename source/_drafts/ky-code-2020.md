---
title: USTC计科复试刷题记录/2020
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:19:40
tags:
	- 刷题
	- 代码
categories:
	- 考研
---

# USTC计科复试刷题记录

[【参考】](https://zdszero.github.io/posts/ustc-test-2021/)

## 2020年

### 1. 最大公约数 【数学】

从标准输入读取两个正整数，求这两个数的最大公约数。

- 枚举法

```c++
#include<cstdio>

int main(){
    int a,b,gcd;
    scanf("%d %d",&a,&b);
    int min=a<b?a:b;// 取其中最小的值
    for(int i=min;i>=1;i--){
        if(b%i==0&&a%i==0)
        {
            gcd=i;
            break;
        }
    }
    printf("%d",gcd);
    return 0;
}
```

- **辗转相除法（欧几里德法）**

```cpp
#include<cstdio>
//辗转相除法 
int gcd(int a,int b) // 需要保证b<=a
{
    if(a%b==0)
        return b;
    else
        return gcd(b,a%b);
}

int main(){
    int a,b;
    scanf("%d %d",&a,&b);
    if(a<b) {
        int temp=a;
        a=b;
        b=temp;
    };
    printf("%d",gcd(a,b));
    return 0;
}
```

- 拓展

最小公倍数为两数的乘积除以最大公约数。

### 2. 第k大的数 【排序】

求长度为n的无序数组中第k大的数。数组从文件`array.in`中读取，首先给出n和k，然后给出数组的n个元素。

```cpp
输入示例:
10 7
6 2 20 99 37 100 33 28 78 44
输出示例:
44
```

- 堆排序   找到第k大的元素即停止排序。

```cpp

```

- 快排   快速排序每一趟可以确定一个元素的位置，所以递归时只需要向一个方向递归即可。

```c++
#include<iostream>
#include<fstream>
#include<vector>
using namespace std;

void partition(vector<int> &nums,int l,int r,int k){
    // 快速排序 每次都能确定一个元素的最终位置 当该回合确定的元素位置为k时返回
    // 从大到小排序
    if (l >= r) return;
    int target=nums[l];
    int i=l,j=r;
    while(i<j){
        // 找第一个小于target的数字
        while(nums[j]<=target&&i<j) j--;
        nums[i]=nums[j];
        while(nums[i]>=target&&i<j) i++;
        nums[j]=nums[i];
    }
    // i==j时跳出循环
    nums[i]=target;
    if(k==i)
        return;
    else if(i<k) // 说明在右边
        partition(nums,i+1,r,k);
    else
        partition(nums,l,i-1,k);

    return ;
}

int main()
{
    int n,k;
    ifstream fs("array.in");
    fs>>n>>k;
    vector<int> nums(n);
    for(int i=0;i<n;i++){
        fs>>nums[i];
        cout<<nums[i]<<" ";
    }
    cout<<endl;
    partition(nums,0,n-1,k-1);
    for(int i=0;i<n;i++){
        cout<<nums[i]<<" ";
    }
    cout<<endl<<nums[k-1];
    return 0;
}
```

### 3. 最大的组合数 【数学】

从文件`number.in`中读取若干个正整数，求出它们最大的组合数，输出到标准输出中。

```c++
输入示例:
123 456 78 782 789
输出示例:
78978782456123
```

```c++
// 求最大组合数
// 从文件number.in中读取若干个正整数，求出它们最大的组合数，输出到标准输出中。
#include<cstdio>
#include<fstream>
#include<iostream>
#include<vector>
#include<string>

using namespace std;

bool compare_str(const string &a, const string &b)
{
    // 比较字符串
    string res1=a+b;
    string res2=b+a;
    if(res1>res2)
        return true;
    else
        return false;
}
void quickSort(vector<string> &v,int l,int r)
{
    if(l>=r) return ;
    string target=v[l];
    int i=l,j=r;
    while(i<j){
        while(i<j&&compare_str(target,v[j])) j--;
        v[i]=v[j];
        while(i<j&&compare_str(v[i],target)) i++;
        v[j]=v[i];
    }
    v[i]=target;
    quickSort(v,l,i-1);
    quickSort(v,i+1,r);
}

int main()
{
    ifstream ifs("number.in");
    vector<string> v;
    string s;
    while (ifs >> s)
        v.push_back(s);
    // 字符串数组排序
    quickSort(v,0,v.size()-1);
    for(const auto &s:v) cout<<s;
    return 0;
}
```

