---
title: 洛谷2-贪心
date: 2020-01-15 20:36:10
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->

Table of Contents
=================

* [1. P1803 凌乱的yyy / 线段覆盖](#1-p1803-凌乱的yyy--线段覆盖)
	* [思路：](#思路)
* [2. P1080 国王游戏](#2-p1080-国王游戏)
	* [思路：](#思路-1)



### 1. P1803 凌乱的yyy / 线段覆盖

#### 思路：

这道题的思路仍然是排序之后扫一遍就可以，这里的核心思想是把n场考试缩减成n-1的子问题，为了使子问题有最优解，就需要确保缩减的时候挑出的第一场考试结束时间尽可能短，所以排序的时候应该以结束时间的早晚进行排序，然后比较：如果某考试的开始时间大于等于上一场选中的考试的结束时间，就可以选择。练手题，两遍AC。

```c++
#include<bits/stdc++.h>

using namespace std;

int n, x, y;
vector<pair<int, int> > nums;
int tail, cnt;

bool cmp(pair<int, int> a, pair<int, int> b){
    return a.second == b.second ? a.first < b.first : a.second < b.second;
}
int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d%d", &x, &y);
        nums.push_back(make_pair(x, y));
    }
    sort(nums.begin(), nums.end(), cmp);
    for(int i = 0; i < n; i++){
//        cout<<nums[i].first<<" "<<nums[i].second<<endl;
        if(nums[i].first >= tail){
            tail = nums[i].second;
            cnt++;
        }
    }
    cout<<cnt<<endl;
    return 0;
}

```

### 2. P1080 国王游戏

#### 思路：

这道题比较难也比较经典，思路非常有趣：

> 首先可以明确的一点是：在某个大臣前面的人无论怎么排都不会影响该大臣得到的钱（乘法交换律）。只有当A和B的相对位置发生变化的时候，A和B得到的钱数才会互相影响（这里相对位置指的是A在B前或者B在A钱）。那么考虑: $ A(a_i, b_i)$ 和$ B(a_j, b_j)$的位置顺序(国王$ (a_0, b_0)$永远在最前面)：
>
> 1. A在B的前面，则设A得到的金币为$ \prod_{a_1}^{a_{i-1} }/b_i$，B得到的金币为$  \prod_{a_1}^{a_{i-1} }*a_i*\prod_{a_{i+1} }^{a_{j-1} }/b_j$， $ans1=max(\prod_{a_1}^{a_{i-1} }/b_i, \ \   \prod_{a_1}^{a_{i-1} }*a_i*\prod_{a_{i+1}}^{a_{j-1}}/b_j)$；
> 2. A在B的后面，则B得到的金币为$ \prod_{a_1}^{a_{i-1} }/b_j$，A得到的金币为$  \prod_{a_1}^{a_{i-1} }*a_j*\prod_{a_{i+1}}^{a_{j-1}}/b_i$，$ans2=max(\prod_{a_1}^{a_{i-1} }*a_j*\prod_{a_{i+1}}^{a_{j-1} }/b_i，\ \ \prod_{a_1}^{a_{i-1} }/b_j)$
>
> 显然，ans1的左项小于ans2的左项，ans1的右项大于ans2的右项，因此实际上比较ans1和ans2的大小就是在比较ans1右项和ans2的左项：
>
> $if\ ans1.right < ans2.left, then\ a_i/b_j<a_j/b_i,\ then\ a_i*b_i<a_j*b_j$
>
> 也即，如果ans1 < ans2，那么就有$ a_i*b_i<a_j*b_j$。且这是充要条件。
>
> 因此，要得到最小的最大值，假设A的a\*b值大于B，那么显然ans1要大于ans2，这个时候为了取到ans2(尽可能小的最大值)，就要让A(a\*b值大的大臣)排在B(a\*b值小)的大臣后面。
>
> 刚才已经说过，每个大臣前面的人怎么排序并不会影响他应得的钱。因此最后的解一定是最优解。

这个思路有了以后，再看这道题，注意数据范围，需要用到高精度：

> 高精度可以通过struct封装常用的乘法、除法、=、<等运算实现，需要注意的几点：
>
> 1. 一般高精度每一个位的整数大小(base取10000)，再大并不会提高效率，并且1000x10000尚未溢出整数范围，对乘法有好处。
> 2. data[N]的最低位data[0]存该大数的“位数”（除data[0]以外的数组长度）。
> 3. 除法的时候需要保证前导零不被输出。
> 4. 输出的时候，先输出最高位(data[data[0]])，然后输出其他位，如果base=10000则应在输出的时候输出“%04d”对应中间位输出长度。

最后代码如下:

```c++
#include<bits/stdc++.h>

using namespace std;

const int N = 10000+5;
int n, x, y;
vector<pair<int, int> > nums;

bool cmp(pair<int, int> a, pair<int, int> b){
    long long xx = a.first*a.second, yy = b.second*b.first;
    return xx < yy;

}

struct BigInt{
    static const int base = 10000;
    int data[N];//data[0] was used to record how many digits there is.

    BigInt(long long num = 0){
        *this = num;
    }
    BigInt operator = (long long num){
        memset(data, 0, sizeof(data));
        for(int i = 1; num > 0; i++){
            data[i] = num % base;
            num /= base;
            data[0]++;
        }
    }
    void multi(int num){
        for(int i = 1; i <= data[0]; i++)
            data[i] *= num;
        for(int i = 1; i < data[0]; i++){
            data[i + 1] += data[i] / base;
            data[i] %= base;
        }
        while(data[data[0]]>=base){
            data[data[0] + 1] += data[data[0]] / base;
            data[data[0]] %= base;
            data[0]++;
        }
    }
    void div(int num){
        int q = 0;
        for(int i = data[0]; i > 0; i--) {
            q *= base;
            q += data[i];
            data[i] = q/num;
            if(data[0] == 0 && data[i] != 0) {
                data[0]=i;
            }
            q %= num;
        }
        for(int i = data[0]; i > 0; i--){
            if(data[i] == 0)
                data[0]--;
            else break;
        }
    }
    void print(){
        printf("%d", data[data[0]]);
        for(int i = data[0]-1; i > 0; i--)
            printf("%04d", data[i]);
        puts("");
    }
    bool compare(BigInt& b){
        if(data[0] > b.data[0])
            return true;
        else if (data[0] < b.data[0])
            return false;
        else{
            for(int i = data[0]; i > 0; i--){
                if(data[i] > b.data[i])
                    return true;
                else if (data[i] < b.data[i])
                    return false;
            }
            return true;
        }
    }
    BigInt (BigInt& b){
        memset(data,0,sizeof(data));
        for(int i = b.data[0];i >=0; i--)
            data[i] = b.data[i];
        return ;
    }
    void mod(int num){
        data[0] = 1;
        data[1] %= num;
        for(int i = 2; i <= data[0]; i++)
            data[i] = 0;
    }
}ans, accu;

int main(){
    scanf("%d", &n);
    for(int i = 0; i <= n; i++){
        scanf("%d%d", &x, &y);
        nums.push_back(make_pair(x,y));
    }
    sort(nums.begin()+1, nums.end(), cmp);
    accu = nums[0].first;
    ans = 0;
    for(int i = 1; i <= n; i++){
        BigInt temp = accu;
        temp.div(nums[i].second);
        if(temp.compare(ans)){
            ans = temp;
        }
        accu.multi(nums[i].first);
    }
    ans.print();
    return 0;
}

```

这道题被高精度卡了好久。。。

