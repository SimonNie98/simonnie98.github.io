---
title: 洛谷1-字符串处理+贪心
date: 2020-01-14 20:36:10
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true

---

<!-- more -->

Table of Contents
=================

* [1. P1071 潜伏者](#1-p1071-潜伏者)
	* [思路：](#思路)
* [2. P1012 拼数](#2-p1012-拼数)
	* [思路：](#思路-1)
* [3. P1090 合并果子](#3-p1090-合并果子)
	* [思路：](#思路-2)
* [4. P1181 数列分段Section I](#4-p1181-数列分段section-i)
	* [思路：](#思路-3)
* [5. P1208 [USACO1.3]混合牛奶 Mixing Milk](#5-p1208-usaco13混合牛奶-mixing-milk)
	* [思路：](#思路-4)
* [6. P1223 排队接水](#6-p1223-排队接水)
	* [思路：](#思路-5)
* [7. P1094 纪念品分组](#7-p1094-纪念品分组)
	* [思路：](#思路-6)



### 1. P1071 潜伏者

#### 思路：

主要是确保明文密文之间存在双射。

```c++
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
int x2y[26], y2x[26];

string a, b, c;
int main(){
    cin>>a;
    cin>>b;
    cin>>c;
    int n = a.size();
    for(int i = 0; i < 26; i++){
        x2y[i] = -1;
        y2x[i] = -1;
    }
    for(int i = 0; i < n; i++){
        if(x2y[b[i]-'A'] == -1 && y2x[a[i]-'A'] == -1 || (x2y[b[i]-'A'] == a[i]-'A' && y2x[a[i]-'A'] == b[i]-'A')){
            x2y[b[i]-'A'] = a[i]-'A';
            y2x[a[i]-'A'] = b[i]-'A';
        } else{
            printf("Failed\n");
            return 0;
        }
    }
    for(int i = 0; i < 26; i++){
        if(x2y[i] == -1 || y2x[i] == -1){
            printf("Failed\n");
            return 0;
        }
    }
    for(int j = 0; j < c.size(); j++){
        printf("%c", y2x[c[j]-'A'] + 'A');
    }
    printf("\n");
    return 0;
}

```

### 2. P1012 拼数

### 思路：

这题的比较简单的做法是通过字符串比较进行排序，我利用了sort和自定义的排序函数完成。

```c++
#include<iostream>
#include<cstdio>
#include<algorithm>
#include <string>

using namespace std;
int n, nums[100];

string to_str(int a){
    string ans;
    while(a){
        ans = char(a%10 + '0') + ans;
        a /= 10;
    }
    return ans;
}

bool cmp(int a, int b){
    string x = to_str(a), y = to_str(b);
    return (x+y) > (y+x);
}

int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", &nums[i]);
    }
    sort(nums, nums+ n, cmp);
    string ans;
    for(int i = 0; i < n; i++)
        ans += to_str(nums[i]);
    cout<<ans<<endl;
    return 0;
}

```

### 3. P1090 合并果子

#### 思路：

合并果子的思路是每次合并的时候找到最小的两个堆合并成到一块，直至合并成1堆了。这题利用了STL的小根堆完成。

```c++
#include<bits/stdc++.h>

using namespace std;

int n, tmp;
int ans, cnt;
std::priority_queue<int, std::vector<int>, std::greater<int> > pq;
int main(){
    scanf("%d", &n);
    for(int i = 0 ; i < n; i++){
        scanf("%d", &tmp);
        pq.push(tmp);
    }
    if (n == 1){
        cout<<0<<endl;
        return 0;
    }
    while(!pq.empty()){
        int top1 = pq.top();
        pq.pop();
        if(pq.empty())
            break;
        int top2 = pq.top();
        pq.pop();
        ans += (top1 + top2);
        pq.push(top1 + top2);
    }
    cout<<ans;
}
```

### 4. P1181 数列分段Section I

#### 思路：

这道题是正整数，只需要扫一遍观察一下就可以。一遍AC。

```c++
#include<bits/stdc++.h>
using namespace std;

int n, m, nums[100010], cnt, sum;
int main(){
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++){
        scanf("%d", &nums[i]);
    }
    for(int i = 0; i < n; i++){
        if(sum + nums[i] <= m){
            sum += nums[i];
        } else{
            cout<<sum<<endl;
            cnt++;
            sum = nums[i];
        }
    }
    cnt++;
    cout<<cnt<<endl;

    return 0;
}
```

### 5. P1208 [USACO1.3]混合牛奶 Mixing Milk

#### 思路：

原则：越便宜的牛奶买越多越好；使用pair<int, int>存储<单价，数量>，对数据按照单价升序、数量降序的顺序排序后依次减过去即可。一遍AC。

```c
#include<bits/stdc++.h>

using namespace std;

int n, m, t1, t2, cnt, fee;
vector<pair<int, int> > milks;
bool cmp(pair<int, int> a, pair<int, int> b){
    return a.first == b.first ? a.second > b.second:a.first < b.first;
}
int main(){
    scanf("%d%d", &n, &m);
    for(int i = 0; i < m; i++){
        scanf("%d%d", &t1, &t2);
        milks.push_back(make_pair(t1, t2));
    }
    sort(milks.begin(), milks.end(), cmp);
    for(int i = 0; i < m && cnt < n; i++){
        if(cnt + milks[i].second > n){
            fee += (n - cnt) * milks[i].first;
            cnt += (n - cnt);
        } else{
            fee += milks[i].second * milks[i].first;
            cnt += milks[i].second;
        }
    }
    cout<<fee<<endl;
    return 0;
}
```

### 6. P1223 排队接水

#### 思路：

还是很简单的，将<order, waiting>排序即可。两遍AC，第一次炸是因为要注意long long，否则会溢出。

```c++
#include<bits/stdc++.h>

using namespace std;

int n, tmp;
long long cnt, tmp2;
vector<pair<int, int> > nums;

bool cmp(pair<int, int> a, pair<int, int> b){
    return a.second == b.second? a.first < b.first : a.second < b.second;
}
int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", &tmp);
        nums.push_back(make_pair(i+1, tmp));
    }
    sort(nums.begin(), nums.end(), cmp);
    for(int i = 0; i < n-1; i++){
        cout<<nums[i].first<<" ";
        tmp2 += nums[i].second;
        cnt += tmp2;
    }
    cout<<nums[n-1].first<<endl;
//    cout<<cnt<<endl;
    printf("%.2f\n", cnt/(n*1.0));
    return 0;
}
```

### 7. P1094 纪念品分组

#### 思路：

因为每组纪念品最多只能组合两件，所以直接排序，对每个最大的纪念品，最好的方法是尽可能搭配一个尽可能小的给他。贪心证明如下：

> 贪心算法：先对数组从小到大排序。用头指针head和为指针tail分别指向首尾。head指向最小元素，tail指向最大元素。
>
> case 1: 如果`nums[head] + nums[tail] > w`, 显然此时`nums[tail]`只能单独一组。
> case 2: 如果`nums[head] + nums[tail] <= w`, 此时分以下情况讨论：
> a. `nums[tail]`单独一组，`nums[head]`单独一组，则最后的结果显然不是最优解(可以将这俩合并)。
> b. `nums[tail]`单独一组，`nums[head] + nums[k]`单独一组，则最后的最优解情况与现在等价，无非就是`nums[tail]`和`nums[k]`谁单着。
> c. `nums[head]`单独一组，`nums[tail]`和其他元素单独一组，同b
> d. `nums[head] + nums[k]`一组， `nums[tail]+nums[m]`一组，此时`nums[k]+nums[m] <= nums[tail]+nums[m]<=w,nums[tail]+nums[head]<=w`,仍然等价于上述最优解。
>
> 某个最优解可以通过上述贪心选择过程得到。
>
> 下面证明 贪心选择加子问题的最优解 为全局最优解。
>
> 设有 `a[i...j]`问题（记为 P）的贪心解为 S， 经过贪心选择之后 子问题 P’的最优解为 S'。 如果贪心选择加子问题 的最优解S'不是 P的最优解。 假设存在一个最优解 Z， 可以通过上面的证明过程，改变解的结构，使之变成一个贪心选择和相同子问题 P'的解 Z‘。 因为 S > Z，并且二者贪心选择所产生的分组数是相同的，所以 S' > Z'， 这与 S'是最优解矛盾。所以贪心选择加子问题的最优解为全局最优解。

一遍AC。

```c++
#include<bits/stdc++.h>
using namespace std;

int w, n, nums[30010], cnt;
int main(){
    scanf("%d", &w);
    scanf("%d", &n);
    for(int i = 0; i < n; i++)
        scanf("%d", &nums[i]);
    sort(nums, nums + n);
    int head = 0, tail = n-1;
    while(head < tail){
        if (nums[tail] + nums[head] <= w){
            head++;
        }
        tail--;
        cnt++;
    }
    if(tail == head)
        cnt++;
    cout<<cnt<<endl;
    return 0;
}

```





