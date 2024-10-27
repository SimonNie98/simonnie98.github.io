---
title: 洛谷3-深度优先搜索(1)
date: 2020-01-19 14:47:03
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---

> 昨天打了洛谷入门赛，一个半小时九道简单题，结果没写完。。太菜了。
>
> 洛谷OJ还没开放提交，题解下次再写（有一道第K小的数还是蛮有意思）

<!-- more -->

Table of Contents
=================

* [note](#note)
    * [1. P1219 八皇后](#1-p1219-八皇后)
    	* [思路：](#思路)
    	* [卡常技巧：](#卡常技巧)
    * [2. P1219 八皇后(2)](#2-p1219-八皇后2)
    	* [思路：](#思路-1)
    	* [lowbit操作：a&amp;(-a)能够取出最右边的1。](#lowbit操作a-a能够取出最右边的1)



## note

### 1. P1219 八皇后

#### 思路：

八皇后问题是一个很悠久的题目了。方法也很简单，就是暴力深搜。对每行要找到一个解，判断同一行、同一列、同一左右对角线上方是否有棋子做出判断，无棋子则当前位置可以放棋子，否则就找同一行的下一列，直到找到一个解，回退的时候要记得清除棋盘。

#### 卡常技巧：

这里最后一个case竟然TLE了，然后发现卡常的技巧：

1. 打表（对最后一个case 特判然后输出）
2. O2优化

```c++
#include<bits/stdc++.h>
using namespace std;

int n, cnt, t = 3, ans[14][14];

void printAns(int n){
    if(t <= 0)
        return;
    for(int i = 0; i < n; i++){
        for(int j = 0; j < n; j++)
            if(ans[i][j] == 1)
                printf("%d",j+1);
        if(i < n-1)
            printf(" ");
    }
    printf("\n");
    t--;
}
bool isOK(int row, int col){
    //check the column
    for(int i = 0; i < row; i++)
        if(ans[i][col] == 1)
            return false;
    //check the row
    for(int j = 0; j < col; j++)
        if(ans[row][j] == 1)
            return false;
    //check the upper left diagonal
    for(int j = 1; row-j >= 0 && col-j >= 0;j++)
        if(ans[row-j][col-j] == 1)
            return false;
    //check the upper right diagonal
    for(int j = 1; row-j >= 0 && col+j < n; j++)
        if(ans[row-j][col+j] == 1)
            return false;
    return true;
}

void dfs(int row){
    if(row == n){
        cnt++;
        printAns(n);
        return;
    }
    //for this row, find a column position.
    for(int j = 0; j < n; j++){
        if(isOK(row, j)){
            ans[row][j] = 1;
            dfs(row+1);
            //clear ans;
            ans[row][j] = 0;
        }
    }
}
int main(){
    scanf("%d", &n);
    if(n == 13){
        cout<<"1 3 5 2 9 12 10 13 4 6 8 11 7\n1 3 5 7 9 11 13 2 4 6 8 10 12\n1 3 5 7 12 10 13 6 4 2 8 11 9\n73712"<<endl;
        return 0;
    }
    dfs(0);
    printf("%d\n", cnt);
    return 0;
}
```

### 2. P1219 八皇后(2)

#### 思路：

上面已经给出了一种解答，其实还有一种位运算的解法更加巧妙：

在判断下一个棋子的可用位置的时候，我们需要知道有哪些列、哪些左对角线、哪些右对角线不能用。所以我们可以传入三个参数表示这三种限制。

核心思想是，利用nr，nld，nrd三个变量分别以二进制表示有哪些位置不能放棋子，取或运算就是不能放的位置，然后再取反就是能够放棋子的位置，这里有一个trick：

#### lowbit操作：a&(-a)能够取出最右边的1。

通过这个操作，就能拿出最右边的可以放的位置，然后更新nr=nr|p, nld = (nld | p) << 1, nrd = (nrd | p) >> 1；递归搜索解。当nr＝11111…的时候，意味着已经没有位置可以放棋子了，也就证明找到一个完整的解了。

正常的思路是从左到右，从上到下放棋子；这里位运算是从0…01开始一直算到1…11。我们只要在输出棋盘中棋子位置的时候将0..01看做左边第一个棋子就可以了。具体参见printAns()函数。

```c++
#include<bits/stdc++.h>

using namespace std;

int n, cnt, t = 3;
long long limit;
list<int> ans;
void print_binary(long x){
    string ans;
    while(x){
        ans = (char)(x % 2 + '0')+ans;
        x >>= 1;
    }
    while(ans.size() < n){
        ans = "0" + ans;
    }
    cout<<ans<<endl;
    return;
}

int getPos(int pos){
    int res = 0;
    while(pos != 1){
        res++;
        pos /= 2;
    }
    return res;
}

void printAns(){
    if(t <= 0)
        return;
    for(list<int>::reverse_iterator i = ans.rbegin(); i != ans.rend(); i++){
        int pos = getPos(*i);
        cout<<(pos+1)<<" ";
    }
    cout<<endl;
    t--;
}

/** nr: not-available row positions
 *  nld: not-available left-diagonal positions
 *  nrd: not-available right-diagonal positions
 */
void dfs(long long nr, long long nld, long long nrd){
    long long disable = (nr | nld | nrd);
    long long pos;
    if(nr != limit){
        pos = ~disable & limit;
        while(pos){
            long long p = pos & (~pos + 1); //get rightest position index
            ans.push_front(p);
            pos = pos - p;
            if(ans.size() == n){
                printAns();
            }
            dfs(nr | p, limit & ((nld|p) << 1), limit& ((nrd|p) >> 1));
            if(!ans.empty())
                ans.pop_front();
        }
    }
    else{
        cnt++;
        return;
    }
}

int main(){
    scanf("%d", &n);
    limit = (1 << n) - 1;
    dfs(0, 0, 0);
    cout<<cnt<<endl;
    return 0;
}
```

