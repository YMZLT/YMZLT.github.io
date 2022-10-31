---
title: USTC计科复试刷题记录/2011
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-31 18:02:46
tags:
	- 刷题
	- 代码
categories:
	- 考研
---



[【参考】](https://zdszero.github.io/posts/ustc-test-2011/)

### 1.进制转换

给两个十进制数，先异或，然后输出其二进制形式。

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int a, b, c;
  cin >> a >> b;
  c = a ^ b;
  int bits[32];
  int N = 0;
  if (c == 0)
    cout << 0;
  while (c) {
    bits[N++] = c & 1;
    c >>= 1;
  }
  for (int i = N-1; i >= 0; i--)
    cout << bits[i];
  return 0;
}
```



### 2. 球的取法

一共有十二个球，其颜色有红、黄、黑三种，红黄黑分别有 x，y，k 个，现在从其中取出八个球，共有多少种取法，输出到文件中？（x，y，k 从键盘输入，同种颜色的球不区分）

```
输入：
3 4 5
输出：
 1. 0红球，3黄球，5黑球
 2. 0红球，4黄球，4黑球
 3. 1红球，2黄球，5黑球
 4. 1红球，3黄球，4黑球
 5. 1红球，4黄球，3黑球
 6. 2红球，1黄球，5黑球
 7. 2红球，2黄球，4黑球
 8. 2红球，3黄球，3黑球
 9. 2红球，4黄球，2黑球
10. 3红球，0黄球，5黑球
11. 3红球，1黄球，4黑球
12. 3红球，2黄球，3黑球
13. 3红球，3黄球，2黑球
14. 3红球，4黄球，1黑球
```

```cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[]) {
  int x, y, k;
  cin >> x >> y >> k;
  int cnt = 0;
  for (int i = 0; i <= 8 && i <= x; i++) {
    for (int j = 0; j <= 8 - i && j <= y; j++) {
      if (8 - i - j <= k) {
        printf("%2d. %d红球，%d黄球，%d黑球\n", ++cnt, i, j, 8 - i - j);
      }
    }
  }
  return 0;
}

```



### 3. 正则表达式

在文件`3.txt`中查看是否有模式 abcde。若有，输出`找到abc*d?e匹配`；若无，则输出`没有找到abc*d?e匹配`，将结果输出到控制台。

- 人工实现一个识别特定正则表达式的有限自动机

```cpp
#include <fstream>
#include <iostream>
#include <sstream>
using namespace std;

string readFile(string filename) {
  ifstream ifs(filename);
  ostringstream oss;
  char c;
  while (ifs >> c) {
    oss.put(c);
  }
  return oss.str();
}

/* abc*d?e */
bool check(const string &s, int idx) {
  int len = s.length();
  if (idx >= len || s[idx++] != 'a') return false;
  if (idx >= len || s[idx++] != 'b') return false;
  while (idx < len && s[idx++] == 'c');
  if (idx < len && s[idx] == 'e') return true;
  if (idx >= len || s[idx++] != 'd') return false;
  if (idx >= len || s[idx] != 'e') return false;
  return true;
}

int main(int argc, char *argv[]) {
  string s = readFile("./3.txt");
  int flag = 0;
  for (int i = 0; i < s.length(); i++) {
    if (check(s, i)) {
      cout << "找到abc*d?e匹配" << endl;
      return 0;
    }
  }
  cout << "没找到abc*d?e匹配" << endl;
  return 0;
}
```

### 4. 二叉树后序遍历

从文件`4.txt`中读入一个二叉树，然后后序遍历该二叉树，将结果输出到文件`4.out`。

```
输入格式：
// 表示结点个数
// 当前结点编号 左孩子编号 右孩子编号
4 
1 2 4 
2 0 3
3 0 0
4 0 0

输出格式：
3 2 4 1
```

```cpp
#include <fstream>
#include <vector>
using namespace std;

int flag = 0;
ifstream ifs("./tree.in");
ofstream ofs("./tree.out");

struct TreeNode {
  TreeNode *lchild, *rchild;
  int val;
  TreeNode(int val): val(val) {}
};

void postOrder(TreeNode *t) {
  if (!t)
    return;
  postOrder(t->lchild);
  postOrder(t->rchild);
  if (flag)
    ofs << " ";
  flag = 1;
  ofs << t->val;
}

int main(int argc, char *argv[]) {
  int n, rootIdx;
  int a, b, c;
  ifs >> n >> rootIdx;
  vector<TreeNode *> nodes(n+1, nullptr);
  ifs.putback(rootIdx+'0');
  while (ifs >> a >> b >> c) {
    if (!nodes[a]) nodes[a] = new TreeNode(a);
    if (!nodes[b] && b != 0) nodes[b] = new TreeNode(b);
    if (!nodes[c] && c != 0) nodes[c] = new TreeNode(c);
    nodes[a]->lchild = nodes[b];
    nodes[a]->rchild = nodes[c];
  }
  TreeNode *root = nodes[rootIdx];
  postOrder(root);
  return 0;
}
```

```cpp
#include<iostream>
#include<fstream>
#include<cstdio>
#include<vector>
using namespace std;
ifstream ifs("input.in");
ofstream ofs("output.out");
struct node
{
    int val;
    node *lchild,*rchild;
    node(int v):val(v),lchild(NULL),rchild(NULL) {}; // 定义构造函数
};
void postOrder(node* root)
{
    if(root==NULL) return;
    postOrder(root->lchild);
    postOrder(root->rchild);
    printf("%d ",root->val);
}
int main()
{
    int n; // 树节点个数
    ifs>>n;
    vector<node*> tree(n+1,NULL); // 保存每行根节点的地址
    int r,a,b;
    int cnt=0;
    int rootId=0;
    while(ifs>>r>>a>>b)
    {
        if(cnt==0) rootId=r; // 记录根节点的位置
        if(tree[r]==NULL) tree[r]=new node(r);
        if(tree[a]==NULL&&a!=0) tree[a]=new node(a);
        if(tree[b]==NULL&&b!=0) tree[b]=new node(b);
        tree[r]->lchild=tree[a];
        tree[r]->rchild=tree[b];
        cnt++;
    }
    postOrder(tree[rootId]);
    return 0;
}
```

