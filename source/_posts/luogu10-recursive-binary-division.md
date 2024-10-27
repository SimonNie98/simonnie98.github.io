---
title: 洛谷10-递推与递归二分
date: 2020-02-27 23:20:58
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1192 台阶问题](#1-p1192-台阶问题)
	* [思路:](#思路)
* [2. P1025 数的划分](#2. p1025 数的划分)
	* [思路：](#思路-1)
	* [方法1:](#方法1)
	* [方法2:](#方法2)
	* [总结：](#总结)




### 1. P1192 台阶问题

#### 思路:

有N级的台阶，你一开始在底部，每次可以向上迈最多K级台阶（最少11级），问到达第N级台阶有多少种不同方式。

这道题有一种解法是斐波那契数列，但是感觉比较难想。参考题解后，觉得DP比较合适。

对于dp[i]来说，它的值等于所有能一步到达它的位置的值之和。

```c++
#include<bits/stdc++.h>
#define mode 100003;
using namespace std;
int n, k, ans[100010];
int main(){
    scanf("%d%d", &n, &k);
    ans[0] = 1, ans[1] = 1;
    for(int i = 2; i <= n; i++){
        for(int j = min(i, k); j >= 1; j--){
            ans[i] += ans[i-j];
            ans[i] %= mode;
        }
    }
    cout<<ans[n]<<endl;
    return 0;
}
```

### 2. P1025 数的划分

#### 思路：

这道题是典型的数的划分问题：将n划分为k个整数的和，比如5, 1, 1和1, 5, 1被认为是同一种划分。

#### 方法1:

状态转移方程是`f[n][k] = f[n-1][k-1]+f[n-k][k]`

这个状态转移方程对应的方法是：

f[n]\[k]表示将n个货物划分为k堆，

首先考虑基础情况，n = 1 或者 k = 1，f[n]\[1] = f[1]\[k] = 1。

然后讨论n < k，f[n]\[k] = 0。

然后是n >= k，分两种情况：

(1) k个数里有1，则此时划分数量等价为f[n-1]\[k-1]。

(2) k个数里没有1，则此时划分数等价为f[n–k]\[k]。

#### 方法2:

首先我们考虑将k堆里每个分配1，则问题等价于n-k（记为m）个货物划分到最多k堆里。

记f[m]\[k]为m个货物最多划分至k堆里的划分个数。

基础情况：m = 1或者 k = 1，f[m]\[1] = f[1]\[k] = 1

当k > m时，f[m]\[k] = f[m]\[m]；

当k <= m时，分两种情况：

(1) 正好分成k堆，则数量等价于f[m-k]\[k]；

(2) 分成小于k堆，则等价于f[m]\[k-1]；

于是得到状态转移方程。

#### 总结：

这类整数划分问题还有一种问法：将n拆成m个正整数的和。本质思想是一样的。要注意分情况讨论。

最后代码：

```c++
#include<bits/stdc++.h>

using namespace std;
int n, k;

int f(int m, int k){
    if(k > m)
        return f(m, m);
    else if(m <= 1 || k <= 1)
        return 1;
    else return f(m-k, k) + f(m, k-1);
}
int g(int n, int k){
    if(k > n)
        return 0;
    else if(k <= 1)
        return 1;
    else return g(n-1, k-1) + g(n-k, k);
}
int main(){
    scanf("%d%d", &n, &k);
//    cout<<f(n-k, k)<<endl;
    cout<<g(n, k)<<endl;
    return 0;
}
```



