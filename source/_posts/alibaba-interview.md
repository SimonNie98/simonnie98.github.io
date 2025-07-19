---
title: 阿里面试
date: 2020-03-23 20:48:33
tags: [Interview]
categories: [Interview]
toc: true
comments: true
---

<!-- more -->

## Table of Contents

* [笔试](#笔试)
	* [1. 笔试题1](#1-笔试题1)
	* [2. 笔试题2](#2-笔试题2)
* [面试](#面试)



## 笔试

### 1. 笔试题1

题目大意是：

给定n，从n个人中找出一个任意人数的小组，再从组里找出一个队长。求不同的小组数量。(mode10e9+7)

首先很明显可以得到公式：
$$
\sum_{k=1}^n{k*\binom{n}{k}=n \times \sum_{k=0}^{n-1}{\binom{n-1}{k}}}=n \times 2^{n-1}
$$
最后我只拿到了70%，正解应该是快速幂。

```c++
#include<bits/stdc++.h>
using namespace std;
#define ll long long
#define mod 1000000007

int n;
ll ans;

int main(){
    scanf("%d", &n);
    ans = n;
    int b = n-1;
    int x = 2;
    while(b){
        if(b&1)
            ans *= x % mod;
        x *= x % mod;
        b >>= 1;
    }
    cout<<ans<<endl;
    return 0;
}

```

### 2. 笔试题2

给定nxm二维地图，#为障碍点，.为通行点，S为开始位置，E为终点。每个时间单位可以向上下左右移动一次（到无障碍点），或者花一个时间单位飞一次，飞的时候是从(x, y)到(x’, y’)，满足关系x+x’ = n+1; y+y’ = m+1；飞行次数最多只有５次；求从起点到终点的最小时间。

目测是DFS，但是我炸了。

```c++

```

## 面试

1. c++ 实现reflection
2. new vs malloc
3. c++的虚函数
4. 计算机网络的层次
5. 前后端分离
6. rest
7. 当场写代码：归并＋二分
8. HTTPS vs HTTP
9. ++i 和i++哪个快
10. 指针和引用的区别