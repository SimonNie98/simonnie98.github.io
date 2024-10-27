---
title: 洛谷5-深度优先搜索(3)
date: 2020-01-23 20:26:11
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1605 迷宫](#1-p1605-迷宫)
	* [思路：](#思路)
* [2. P1040 加分二叉树](#2-p1040-加分二叉树)
	* [思路：](#思路-1)
	* [二叉树的前序输出：](#二叉树的前序输出)
* [3. P1092 虫食算](#3-p1092-虫食算)
	* [思路：](#思路-2)



### 1. P1605 迷宫

#### 思路：

这道题是经典的迷宫的题目，只需要dfs暴搜即可。注意状态的修改与还原。

```c++
#include<bits/stdc++.h>

using namespace std;

int n, m, t, sx, sy, fx, fy, tmpx, tmpy;
int stop[26][26], vis[26][26], ans;
int dire[4][2] = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
void dfs(int x, int y){
//    cout<<"x:"<<x<<" "<<y<<endl;
    if(x == fx && y == fy){
        ans++;
        return;
    }

    for(int i = 0; i < 4; i++){
        int xx = x + dire[i][0], yy = y + dire[i][1];
        if(xx <= 0 || xx > n || yy <= 0 || yy > m)
            continue;
        if(vis[xx][yy] == 0 && stop[xx][yy] == 0){
            vis[x][y] = 1;
            dfs(xx, yy);
            vis[x][y] = 0;
        }
    }
}
int main(){
    scanf("%d%d%d", &n, &m, &t);
    scanf("%d%d%d%d", &sx, &sy, &fx, &fy);
    for(int i = 0; i < t; i++){
        scanf("%d%d", &tmpx, &tmpy);
        stop[tmpx][tmpy] = 1;
    }
    dfs(sx, sy);
    cout<<ans<<endl;
    return 0;

}

```

### 2. P1040 加分二叉树

#### 思路：

这道题分为两个子问题：

（1）对于给定节点权值的中序遍历二叉树，怎样找到符合规定计算方法的得分最高的二叉树，输出这个分值

（2）输出这个二叉树的对应的前序遍历

解法：

采用带记忆的搜索dp，

变量dp[i]\[j]表示从i到j的最大值，很显然:

dp[i]\[j] = 1,		 if i == j

​			 = max(dp[i]\[k]*dp[k+1]\[j] + dp[k]\[k]) 	for k in [i, j]

dp[i]\[i] = score[i]，即每个叶子节点的分数是它本身的分数。

与此同时，保存一个变量root[i]\[j]表示从i到j这段子树的根节点。显然root[i]\[i]=[i]。

#### 二叉树的前序输出：

二叉树的前序输出就是先输出根节点再输出左右节点。代码的递归写法很简单。

最终代码如下：

```c++
#include<bits/stdc++.h>
using namespace std;

#define ll long long


int n, score[50], root[50][50];
ll ans, dp[50][50];
// dp[i][j] means the score from i to j;
// root[i][j] means the root of sub-tree from i to j;

int search_tree(int left, int right){
    if(dp[left][right] > 0)
        return dp[left][right];
    if(left == right)
        return score[left];
    if(left > right)
        return 1;
    for(int i = left; i <= right; i++){
        ll now = 1ll * search_tree(left, i-1) * search_tree(i+1, right) + dp[i][i];
        if(now > dp[left][right]){
            dp[left][right] = now;
            root[left][right] = i;
        }
    }
    return dp[left][right];
}

void preorder(int left, int right){
    if(left > right)
        return;
    if(left == right){
        cout<<left+1<<" ";
        return;
    }
    else
        cout<<root[left][right]+1<<" ";
    preorder(left, root[left][right] - 1);
    preorder(root[left][right]+1, right);
}
int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", &score[i]);
        dp[i][i] = score[i];
        root[i][i] = i;
    }
    cout<<search_tree(0, n-1)<<endl;
    preorder(0, n-1);
    return 0;
}
```

### 3. P1092 虫食算

#### 思路：

这道题仍然是暴搜，但是会TLE，关键在于剪枝的技巧。

先上一种只能过80%数据的解法，正解需要用到高斯消元。回头再补。

```c++
#include<bits/stdc++.h>

using namespace std;
int n;
char a[26], b[26], c[26];
int ans[26], used[26], record[26];
char order[26];
bool ok(int type){
    if(type == 1){// strict check, missing is not allowed.
        int ad = 0;
        for(int i = n-1; i >= 0; i--){
            if(ans[a[i]-'A'] == -1 || ans[b[i] - 'A'] == -1 || ans[c[i]-'A'] == -1)
                return false;
            if((ans[a[i]-'A'] + ans[b[i] - 'A'] + ad) % n != ans[c[i]-'A'])
                return false;
            ad = (ans[a[i]-'A'] + ans[b[i] - 'A'] + ad) / n;
        }
        return true;
    }
    else if(type == 0){// not-strict check
        for(int i = n-1; i >= 0; i--){
            if(ans[a[i]-'A'] == -1 || ans[b[i] - 'A'] == -1 || ans[c[i]-'A'] == -1){
                continue;
            }
            if((ans[a[i]-'A'] + ans[b[i] - 'A'])%n != ans[c[i]-'A'] && (ans[a[i]-'A'] + ans[b[i] - 'A'] + 1)%n != ans[c[i]-'A'])
                return false;
        }
        return true;
    }
}

void print(){
    if(!ok(1))
        return;
    for(int i = 0; i < n; i++)
        cout<<ans[i]<<" ";
    cout<<endl;
    exit(0);
}


void dfs(int index){
    if(index >= n){
        print();
        return;
    }
    for(int i = 0; i < n; i++){
        if(used[i] == 1)
            continue;
        ans[order[index]-'A'] = i;
        if(ok(0)){
            used[i] = 1;
            dfs(index+1);
            used[i] = 0;
        }
    }
    ans[order[index] - 'A'] = -1;
}
int main(){
    scanf("%d", &n);
    scanf("%s%s%s", a, b, c);
    for(int i = n-1, j = 0; i >= 0; i--){
        if(!record[a[i]-'A']) order[j++] = a[i], record[a[i] -'A'] = 1;
        if(!record[b[i]-'A']) order[j++] = b[i], record[b[i] -'A'] = 1;
        if(!record[c[i]-'A']) order[j++] = c[i], record[c[i] -'A'] = 1;
        //initialization
        ans[a[i] - 'A'] = -1;
        ans[b[i] - 'A'] = -1;
        ans[c[i] - 'A'] = -1;
    }
    dfs(0);
    return 0;
}
```





