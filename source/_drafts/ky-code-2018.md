---
title: USTC计科复试刷题记录/2018
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:19:30
tags:
	- 刷题
	- 代码
categories:
	- 考研
---

# USTC计科复试刷题记录

[【参考】](https://zdszero.github.io/posts/ustc-test-2018/)

## 2018年

### 1. 计算日期【数学】

2000年1月1日是星期六，现在给你任意一个天数n，代表其距离2000年1月1日有n天，让你求该天是几年几月几日，星期几。比如n=2，输出“2000-1-3 Monday”。（记得注意闰年有366天，而且二月有29天）

```
输入：2
输出：2000-1-3 Monday
```

```cpp
#include<cstdio>
#include<iostream>
using namespace std;

static string week_name[] = {"Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Firaday"};
static int months_days[12]={31,28,31,30,31,30,31,31,30,31,30,31};
static int year=2000,month=1,day=1; // 初始日期

//2000年1月1日是星期六
//2000-1-3 Monday

int getYear(int year)
{
    // 闰年
    if((year%100==0&&year%400==0)||(year%100!=0&&year%4==0)){
            return 366;
    }
    return 365;
}

int main()
{
    int n;
    cin>>n;

    string week=week_name[n%7];
    while(n>=getYear(year)){
        n-=getYear(year);
        year++;
    }
    months_days[1]=(getYear(year)==365?28:29);

    while(n>=months_days[month-1])
    {
        n-=months_days[month-1];
        month++;
    }
    day+=n;
    printf("%d-%d-%d %s",year,month,day,week.c_str());
    return 0;
}
```

### 2. 签到时间【哈希表】

文件`input2.txt`中存储着机房中的m条签到记录，其中每条签到记录由学生的学号、签到时间、签退时间组成。若签退时间小于签到时间，则说明跨天。一人可以多次签到，但多次签到的时间没有重合。

文件`input2.txt`中第一行是m，接下来是m行签到记录，要求给出在机房中停留时间最长的学生学号，并输出到标准输出。

```cpp
输入示例:
3
SA0010001 13:00 16:39
SA0010101 07:22 22:01
SA0010111 12:00 11:56
输出示例:
SA0010111
```

```cpp
#include<iostream>
#include<fstream>
#include<unordered_map>

using namespace std;

int getStayTime(string st,string et)
{
    int h=(st[0]-'0')*10+st[1]-'0';
    int m=(st[3]-'0')*10+st[4]-'0';
    int s=h*60+m; // 转换成分钟表示

    h=(et[0]-'0')*10+et[1]-'0';
    m=(et[3]-'0')*10+et[4]-'0';
    int e=h*60+m;

    return s<e?e-s:e+24*60-s; // 判断是否当天进入或隔天
}

unordered_map<string,int> records; // 记录所有学生出入记录

int main()
{
    ifstream ifs("input2.txt");
    int m;
    ifs>>m;
    for(int i=0; i<m; i++)
    {
        string id,st,et;
        ifs>>id>>st>>et;
        auto iter = records.find(id);
        if(iter==records.end()) // 未找到key 则返回末尾的迭代器
            records.emplace(id,getStayTime(st,et));
        else
            iter->second+=getStayTime(st,et);
    }

    int stay=0;
    string id;
    // 遍历哈希表，找值最大的返回
    for(auto iter=records.begin();iter!=records.end();iter++){
        if(iter->second>stay)
        {
            id=iter->first;
            stay=iter->second;
        }
    }
    cout<<id;
    return 0;
}
```

### 3. 邮票面值【BFS】【完全背包问题】

给定n种面值的邮票，问用这n种面值的邮票能不能凑够总面值m，如果能的话输出所需要的**最少邮票个数**，不能的话输出0。

邮票的信息在文件`input3.txt`中给出，第一行给出m和n的值，第二行给出n种邮票的面值，将结果输出到标准输出。

```cpp
输入示例:
10 3
1 2 4
输出示例:
3
```

```cpp
#include<fstream>
#include<cstdio>
#include<queue>
#include<vector>

using namespace std;
ifstream ifs("input.in");
int m=0,n=0;
int BFS(vector<int> &cards)
{
    priority_queue<int> Q; // 大顶堆
    for(int i=0;i<n;i++) // 初始化优先队列
    {
        Q.push(cards[i]);
    }
    int cnt=0; // 记录当前使用的邮票数目
    while(!Q.empty()){
        int cur=Q.top();
        Q.pop();
        cnt++;
        if(cur==m) return cnt;
        for(int i=0;i<n;i++)
        {
            if(cards[i]+cur<=m)  // 将符合要求的直接加入优先队列
                Q.push(cards[i]+cur);
        }
    }
    return 0; // 都尝试过还没找到则返回0
}
int main()
{

    ifs>>m>>n;
    vector<int> cards(n);
    for(int i=0;i<n;i++)
        ifs>>cards[i];
    printf("%d",BFS(cards));
    return 0;
}

```

### 4. 最长公共子序列【DP】

[leetcode](https://leetcode-cn.com/problems/longest-common-subsequence/)

求两个字符串的最长公共子序列的长度，字符串在文件`input4.txt`中给出，将结果输出到标准输出。

```cpp
输入示例:
abcfac abcdfd
programming contest
abc mmp
输出示例:
4
2
0
```

> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
> 两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

<img src="../../../其他/中科大复试代码/复试机试.assets/image-20220227145502041.png" alt="image-20220227145502041" style="zoom:67%;" />

- 迭代法

```cpp
#include<iostream>
#include<fstream>
#include<cstring>
using namespace std;

// 二维矩阵法
int solve(string text1,string text2)
{
    int m=text1.size(),n=text2.size();
    int dp[m+1][n+1];
    memset(dp,0,sizeof(dp)); // 初始化为0
    for(int i=0;i<m;i++)
    {
        for(int j=0;j<n;j++)
        {
            if(text1[i]==text2[j])
                dp[i+1][j+1]=dp[i][j]+1;
            else
                dp[i+1][j+1]=max(dp[i][j+1],dp[i+1][j]);
        }
    }
    return dp[m][n];
}
int main()
{
    ifstream ifs("input4.txt");
    string s1,s2;
    while(ifs>>s1>>s2)
    {
        cout<<solve(s1,s2)<<endl;
    }
    return 0;
}
```

- 递归法

```cpp
#include<iostream>
#include<fstream>

using namespace std;
// 递归法
int solve(const string &text1,const string &text2,int i,int j)
{
    if(i==-1||j==-1) return 0;

    if(text1[i]==text2[j])
        return solve(text1,text2,i-1,j-1)+1;
    else
    {
        int res1=solve(text1,text2,i-1,j);
        int res2=solve(text1,text2,i,j-1);
        return max(res1,res2);
    }
}
int main()
{
    ifstream ifs("input4.txt");
    string s1,s2;
    while(ifs>>s1>>s2)
    {
        //cout<<s1<<" "<<s2<<endl;
        cout<<solve(s1,s2,s1.size()-1,s2.size()-1)<<endl;
    }
    return 0;
}
```

