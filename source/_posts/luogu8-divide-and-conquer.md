---
title: 洛谷8-分治算法
date: 2020-02-11 15:43:55
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1226 【模板】快速幂||取余运算](#1-p1226-模板快速幂取余运算)
	* [思路：](#思路)
	* [复杂度：](#复杂度)
	* [代码：](#代码)
* [2. P1010 幂次方](#2-p1010-幂次方)
	* [打表版](#打表版)
	* [递归版](#递归版)
* [3. P1908 逆序对](#3-p1908-逆序对)
	* [快读模板：](#快读模板)
	* [归并排序：](#归并排序)



### 1. P1226 【模板】快速幂||取余运算

#### 思路：

这道题就是经典的快速幂。快速幂的思路是，对于`a^b`这种，把b写成2进制，然后把`a^b`分解成$ a^{x_1*2^{0} + x_2*2^{1} + ...+x_n*2^{n-1}}$其中$x_1, x_2,...，x_n为0或者1$。这样一来，就可以通过位运算依次取出二进制表示的每一位，然后不断地相乘即可。

这道题要注意三点，一是最后的输出形式有要求。二是在计算过程中要不断取余以防止整数溢出。三是对于b=0的情况要特判。

#### 复杂度：

快速幂的复杂度从N降到$log_2N$。

#### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;

int b, p, k;

int fast_pow(int b, int p, int k){
    if(!p) return b % k;
    long long res = 1;
    long long x = b;
    while(p){
        if(p&1){
            res = res * x % k;
        }
        x = x * x % k;
        p >>= 1;
    }
    return res;
}
int main(){
    scanf("%d%d%d", &b, &p, &k);
    cout<<b<<"^"<<p<<" mod "<<k<<"="<<fast_pow(b, p, k)<<endl;
    return 0;
}

```

### 2. P1010 幂次方

这种递归的题目，，，肯定是用打表做啦。。。（开玩笑）

一看数据最多也就`2^16以下，论打表的可行性。

`dict`数组中，`dict[i]`存储的是`2^i`的表示方式。然后对输入的n按位分解再输出就可以了。

#### 打表版

```c++
#include<bits/stdc++.h>
using namespace std;
int n;
int nums[16];
string dict[16] = {
    "2(0)",
    "2",
    "2(2)",
    "2(2+2(0))",
    "2(2(2))",
    "2(2(2)+2(0))",
    "2(2(2)+2)",
    "2(2(2)+2+2(0))",
    "2(2(2+2(0)))",
    "2(2(2+2(0))+2(0))",
    "2(2(2+2(0))+2)",
    "2(2(2+2(0))+2+2(0))",
    "2(2(2+2(0))+2(2))",
    "2(2(2+2(0))+2(2)+2(0))",
    "2(2(2+2(0))+2(2)+2)",
    "2(2(2+2(0))+2(2)+2+2(0))",
};
void getPow(int n){
    int index = 0;
    int pow = 0;
    while(n){
        if(n&1)
            nums[index++] = pow;
        n >>= 1;
        pow++;
    }
}
string getAns(){
    string res = "";
    for(int i = 15; i >= 0; i--){
        if(nums[i] == -1)
            continue;
        if(nums[i] > -1){
            res += dict[nums[i]];
            if(i > 0)
                res += "+";
        }


    }
    return res;
}
int main(){
    scanf("%d", &n);
    memset(nums, -1, sizeof(nums));
    getPow(n);
    cout<<getAns()<<endl;
    return 0;
}

```

#### 递归版

调用这个函数即可，不多解释。

```c++
string recur(int n){
    if(n == 0)
        return "0";
    string res = "";
    int i = 0;
    while(n){
        if(n&1){
            if(i == 1)
                res = string("2") + (res == "" ? "" : "+") + res;
            else
                res = "2(" + recur(i) + ")" + (res == "" ? "" : "+") + res;
        }
        i++;
        n >>= 1;
    }
    return res;
}
```

### 3. P1908 逆序对

这道题是想求逆序对的个数，`N^2`肯定会超时。数据量非常大。

第一种解法是归并排序(`NlogN`)，在归并排序的过程中（默认升序），对左区间中的某个元素，如果它大于右边某个元素，那么左边区间剩下的所有元素都大于右边这个元素，答案就可以更新。

归并完成以后，答案也就出来了。

#### 快读模板：

```c++
inline int read(int &res){
    char ch = ' ';
    int w = 1;
    while(ch != '-' && (ch < '0' || ch > '9'))
        ch = getchar();
    if(ch == '-')
        w = -1;
    else
        res = ch - '0';
    for(ch = getchar(); ch >= '0' && ch <= '9'; ch = getchar())
        res = res * 10 + ch - '0';
    res *= w;
    return res;
}
```



#### 归并排序：

```c++
#include<bits/stdc++.h>

using namespace std;
int n, nums[500001], cp[500001];
long long ans;

inline int read(int &res){
    char ch = ' ';
    int w = 1;
    while(ch != '-' && (ch < '0' || ch > '9'))
        ch = getchar();
    if(ch == '-')
        w = -1;
    else
        res = ch - '0';
    for(ch = getchar(); ch >= '0' && ch <= '9'; ch = getchar())
        res = res * 10 + ch - '0';
    res *= w;
    return res;
}

void merge_sort(int left, int right){
    if(left == right)
        return;
    int mid = (left + right)/2;
    merge_sort(left, mid);
    merge_sort(mid+1, right);
    int i = left, j = mid+1, k = left;
    while(i <= mid && j <= right){
        if(nums[i] <= nums[j])
            cp[k++] = nums[i++];
        else
            cp[k++] = nums[j++], ans += mid - i + 1;
    }
    while(i <= mid)
        cp[k++] = nums[i++];
    while(j <= right)
        cp[k++] = nums[j++];
    for(int p = left; p <= right; p++)
        nums[p] = cp[p];
}
int main(){
    read(n);
    for(int i = 0; i < n; i++)
        read(nums[i]);
    merge_sort(0, n-1);
    cout<<ans<<endl;
    return 0;
}
```

第二种解法是树状数组（改日更新）

洛谷该章节的第四题有点恶心，不想写。。

