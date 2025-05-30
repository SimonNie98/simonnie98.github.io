---
title: 洛谷9-简单数学问题
date: 2020-02-12 01:55:41
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1088 火星人](#1-p1088-火星人)
	* [思路：](#思路)
	* [STL版本](#stl版本)
* [2. P1045 麦森数](#2-p1045-麦森数)
	* [思路：](#思路-1)
	* [代码：](#代码)
* [3. P1403 [AHOI2005]约数研究](#3-p1403-ahoi2005约数研究)
	* [思路：](#思路-2)
	* [代码：](#代码-1)
* [4. P1017 进制转换](#4-p1017-进制转换)
	* [思路：](#思路-3)
* [5. P1147 连续自然数和](#5-p1147-连续自然数和)
	* [思路：](#思路-4)
	* [1. 枚举左右端点](#1-枚举左右端点)
	* [2. 前缀和数组](#2-前缀和数组)
	* [3. 解二次方程](#3-解二次方程)
	* [代码：](#代码-2)
* [6. P1029 最大公约数和最小公倍数问题](#6-p1029-最大公约数和最小公倍数问题)
	* [思路：](#思路-5)
	* [tips: P*Q = x * y](#tips-pq--x--y)



### 1. P1088 火星人

#### 思路：

这道题的本质是将给定排列顺序的数组按照字典序改变排列顺序m次后输出。

STL版本就是调用`next_permutaion(nums, nums+n)`函数m次就可以了。

#### STL版本

```c++
#include<bits/stdc++.h>

using namespace std;
int n, m, nums[100010];

inline int read(int &res){
    int w = 1;
    char ch = ' ';
    while(ch != '-' && (ch < '0' || ch >'9'))
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
int main(){
    read(n), read(m);
    for(int i = 0; i < n; i++)
        read(nums[i]);
    while(m--)
        next_permutation(nums, nums+n);
    for(int i = 0; i < n; i++)
        cout<<nums[i]<<" ";
    return 0;
}


```

非STL版本比较简单的思路就是自己手写一个排列顺序的函数。

### 2. P1045 麦森数

#### 思路：

这道题要我们输出两个问题的解：

1. $2^p-1$的位数；
2. $2^p-1$后500位的具体数值。

此处P的范围在1000到3100000。我们只能一个一个问题来解决。

首先，考虑到`2^p`的末尾不可能是0，所以`2^p-1`与`2^p`位数一定是相同的。这时我们很容易就得到`2^p`的位数是$log_{10}{2^p}+1=1+p*log_{10}2$。取整即可。

其次，这道题首先会想到快速幂。但是高精度的快速幂真的好难啊，，，好在我们只需要考虑后500位的数据。

最后的结果如下：

#### 代码：

```c++
#include<bits/stdc++.h>

const int N = 1010;
const int BASE = 10;
using namespace std;

int p, nums[N], cp[N], res[N], ans[N];

void multi_array(int nums1[N], int nums2[N]){
    memset(res, 0, sizeof(res));
    res[0] = 1000;
    for(int i = 1; i <= 500; i++)
        for(int j = 1; j <= 500; j++){
            res[i+j-1] += nums1[i] * nums2[j];
        }
    for(int i = 1; i < res[0]; i++){
        res[i+1] += res[i] / BASE;
        res[i] %= BASE;
    }
    while(res[0] <= N && res[res[0]] >= BASE){
        res[res[0] + 1] += res[res[0]] / BASE;
        res[res[0]] %= BASE;
        res[0]++;
    }
    memset(nums1, 0, sizeof(nums1));
    for(int i = 0; i <= res[0]; i++)
        nums1[i] = res[i];
}

void fast_pow(int nums[N], int p, int ans[N]){
    if(p == 0){
        memset(ans, 0, sizeof(0));
        ans[0] = ans[1] = 1;
        return;
    }
    if(p == 1){
        for(int i = 0; i <= nums[0]; i++)
            ans[i] = nums[i];
        return;
    }
    memset(ans, 0, sizeof(ans));
    ans[0] = ans[1] = 1;
    while(p){
        if(p & 1){
            multi_array(ans, nums);
        }
        p >>= 1;
        multi_array(nums, nums);
    }
    return;
}


int main(){
    scanf("%d", &p);
    cout<<(int)(p*log10(2) + 1)<<endl;
    nums[0] = 1, nums[1] = 2;
    fast_pow(nums, p, ans);
    ans[1] -= 1;
    for(int i = 500; i >= 1; i--)
        if(i != 500 && i % 50 == 0)
            printf("\n%d",ans[i]);
        else printf("%d",ans[i]);
    return 0;
}

```

注意这里只处理后500位。

### 3. P1403 [AHOI2005]约数研究

#### 思路：

把每个正数N的约数的个数用 f(N) 来表示，求$\sum_{i=1}^{n}f(i)$的值。

考虑思路：逆向思维，如果我们去枚举1到N的每个数的约数个数相加，必然时间复杂度要爆炸。反过来想，对于一个数N，它的约数必定在1到N之间。于是，我们遍历1到N之间的每个数，每个数都必然属于一个或多个数的约数集合中。比如1，它是1,2,3…N所有数的约数，而2则是2,4,6..的约数。对每个正整数$i$，它属于$N/i$个正整数的约数集合中。所以我们枚举每一个数，累加它属于多少个正整数的约数中即可得到答案。

#### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;

int n;

int main(){
    scanf("%d", &n);
    int ans = 0;
    for(int i = 1; i <= n; i++)
        ans += n/i;
    cout<<ans<<endl;
    return 0;
}

```

### 4. P1017 进制转换

#### 思路：

这道题给定一个十进制整数，给定一个负数，将这个十进制整数转换为以这个负数为基本进制的表示形式。

这道题最开始想歪了，我想的是从高位依次求到低位，十分复杂。实际上的做法是从低位不断求余数和商，将得到的的商继续递归地去求余数。

比如：101在十进制中，第一次求余数是1，商是10；再把10对10求余数是0，商是1；再把1对10求余是1，商是0；倒过来，101在十进制中的表示就是101。

但是在负数的情况下要注意一点，我们希望余数是正数（最后表示的结果每一位都是正数），否则如果是负数的话就可以继续减去一个除数，商比原来加1。

> (商+1)×除数+余数-除数=原来的被除数。

最终代码如下：

```c++
#include<bits/stdc++.h>

using namespace std;

int n, base, pos;
char nums[100];
char dict[20] = {'0', '1', '2', '3', '4', '5','6', '7', '8', '9',
                 'A', 'B','C', 'D', 'E', 'F', 'G', 'H', 'I', 'J'};


void transfer(int n, int base){
    if(n == 0)
        return;
    int remain = n % base;
    if(remain < 0){
        remain -= base;
        n += base;
    }
    int div= n / base;
    nums[pos++] = dict[remain];
    transfer(div, base);
}
int main(){
    scanf("%d%d", &n, &base);
    transfer(n, base);
    cout<<n<<"=";
    for(int i = pos-1; i >= 0; i--)
        cout<<nums[i];
    cout<<"(base"<<base<<")"<<endl;
    return 0;
}
```

### 5. P1147 连续自然数和

#### 思路：

这道题有很多做法：

##### 1. 枚举左右端点

枚举左右端点，得到两个端点之间的自然数和，与目标值进行比较，然后更新左右端点。O(N)复杂度。

##### 2. 前缀和数组

前缀和数组也可以通过枚举左右端点的方式来枚举前缀和。原理是一样的。难点在于数据大小的限制，有可能造成数组访问越界和加和溢出的问题。因此最好使用`long long`进行求和。

##### 3. 解二次方程

因为sum(L, R) = (L+R)(R-L+1)/2，所以可以令sum(L, R) = M，令K1 = L+R，K2 = R - L + 1，解出L和R关于K1和K2的关系式。**L=(K2-K1+1)/2, R=(K1+K2-1)/2**。不过有一种特殊情况，就是L=R的情况，这种情况是不允许的。然后遍历即可。

#### 代码：

```c
#include<bits/stdc++.h>

using namespace std;
long long n, nums[2000001];
int getArray(int n){
    for(int i = 1; i <= 2000001 && nums[i] <= 2 * n; i++){
        nums[i] = nums[i-1] + i;
    }
}

void findAns(int n){
    for(int left = 0, right = 0; left < n/2;){
        if(nums[right] - nums[left] < n)
            right++;
        else if(nums[right] - nums[left] == n){
            cout<<left+1<<" "<<right<<endl;
            left++;
        }else{
            left++;
        }
    }
}
int main(){
    scanf("%d", &n);
    getArray(n);
    findAns(n);
    return 0;
}
```

### 6. P1029 最大公约数和最小公倍数问题

#### 思路：

这道题给出x，y，要求符合以x为最大公约数以y为最小公倍数的数对P，Q的个数。

#### tips: P*Q = x * y

这里一定要注意，两个数的乘积就是这两个数的最大公约数和最小公倍数的乘积。

然后可以暴力枚举即可。

```c++
#include<bits/stdc++.h>

using namespace std;
#define ll long long

int x, y, ans;

ll gcd(ll m, ll n){
    if(n == 0) return m;
    return gcd(n, m%n);
}
int main(){
    scanf("%d%d", &x, &y);
    ll p = 1ll * x * y;
    for(int i = 1; i <= sqrt(p); i++){
        if(p % i == 0 && gcd(i, p/i) == x){
            ans+=2;
            if(1ll*i*i == p)
                ans--;
        }
    }
    cout<<ans<<endl;
    return 0;
}

```

注意gcd函数的写法！！！！



