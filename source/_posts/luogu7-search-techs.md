---
title: 洛谷7-带有技巧的搜索
date: 2020-02-07 18:00:14
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1118 [USACO06FEB]数字三角形](#1-p1118-usaco06feb数字三角形)
	* [思路：](#思路)
	* [暴力版：](#暴力版)
	* [正解版：](#正解版)
* [2. P1434 [SHOI2002]滑雪](#2-p1434-shoi2002滑雪)
	* [思路：](#思路-1)
* [3. P1433 吃奶酪](#3-p1433-吃奶酪)
	* [思路：](#思路-2)
	* [DFS](#dfs)
	* [状态压缩DP](#状态压缩dp)
* [4. P1074 靶形数独](#4-p1074-靶形数独)
	* [思路：](#思路-3)
	* [剪枝技巧：](#剪枝技巧)
	* [代码：](#代码)



### 1. P1118 [USACO06FEB]数字三角形

#### 思路：

这道题有点技巧，需要先举例得到规律。

首先，如果不考虑任何技巧，无非就是`n!`种排列（肯定TLE），然后对每种排列迭代n次进行求和，然后去判断是否与目标值相等。

可以先在纸上尝试一下比如`a1, a2, a3, a4`四个数，最后的和为`1*a1 + 3*a2 + 3*a3 + 1*a4`。再尝试别的n值的时候发现，最后的和的各项系数显然呈现杨辉三角的规律。得到这个规律之后，对于一个特定的的n，我们很快就能直接求出最后各项系数。

到这一步，我们又可以利用STL的next_permutation函数对数列进行排列，对每个排列依次求和判断就可以了。这个解法可以拿到80分。因为穷举`n!`的排列在题中肯定超时了。

#### 暴力版：

```c++
#include<bits/stdc++.h>

using namespace std;
int n, sum, coff[15], nums[15], used[15], add;

void getCoff(int n){
    for(int i = 0; i < n; i++){
        coff[0] = 1;
        int old = coff[0];
        for(int j = 1; j <= i; j++){
            int tmp = coff[j];
            coff[j] += old;
            old = tmp;
        }
    }
}

void getNum(int n){
    for(int i = 0; i < n; i++)
        nums[i] = i+1;
}

void print(int nums[], int n){
    for(int i = 0; i < n; i++)
        cout<<nums[i]<<" ";
    cout<<endl;
}

int getSum(int coff[], int nums[], int n){
    int res = 0;
    for(int i = 0; i < n; i++)
        res += (coff[i] * nums[i]);
    return res;
}

int main(){
    scanf("%d%d", &n, &sum);
    getCoff(n);
    getNum(n);
    do {
        int add = getSum(coff, nums, n);
        if(add == sum){
            print(nums, n);
            return 0;
        }

    } while(next_permutation(nums, nums+n));

    return 0;
}

```

于是我们需要使用dfs搜索可能的解，因为原题是说如果有多种解答需要输出字典序最小的，所以我们要按照字典序去搜索。因此这里我们考虑从左到右对每个位置，从小到大地查找`1~n`中可以用的数字。找到以后标记该数字已被使用，然后继续查找下一个位置。注意dfs在搜索的时候需要回退状态。只有当找到正解的时候可以不用回退状态（也可以回退，找到正解时将正解写入额外的数组）。最后的解法如下：

#### 正解版：

```c++
#include<bits/stdc++.h>

using namespace std;
int n, sum, coff[15], nums[15], used[15], add;

void getCoff(int n){
    for(int i = 0; i < n; i++){
        coff[0] = 1;
        int old = coff[0];
        for(int j = 1; j <= i; j++){
            int tmp = coff[j];
            coff[j] += old;
            old = tmp;
        }
    }
}

void print(int nums[], int n){
    for(int i = 0; i < n; i++)
        cout<<nums[i]<<" ";
    cout<<endl;
}

int getSum(int coff[], int nums[], int n){
    int res = 0;
    for(int i = 0; i < n; i++)
        res += (coff[i] * nums[i]);
    return res;
}
bool dfs(int index, int num, int add){
    if(add > sum)
        return false;
    if(index == n){
        if(add == sum)
            return true;
        return false;
    }
    used[num] = 1;
    for(int i = 1; i <= n; i++){
        if(used[i] == 1)
            continue;
        if(dfs(index + 1, i, add + coff[index] * i)){
            nums[index] = i;
            return true;
        }
    }
    used[num] = 0;
    return false;

}
int main(){
    scanf("%d%d", &n, &sum);
    getCoff(n);
    if(dfs(0, 0, 0))
        for(int i = 0; i < n; i++)
            cout<<nums[i]<<" ";
    return 0;


    return 0;
}

```

### 2. P1434 [SHOI2002]滑雪

#### 思路：

2D矩阵的最长下降路线。【在POJ做过】

这道题需要使用记忆化搜索+剪枝来做，简单的DFS必然会超时。

在考虑解题的时候，肯定是对每个点都DFS一次，得到在该点能滑行的最长距离。但是这样会炸时间。所以我们要想办法剪枝从而避免重复搜索。

首先我们可以保证，因为对每个点都DFS一次，所以被DFS过的点一定已经找到了从该点能向下滑行的最大距离。这样我们在DFS的过程中，如果碰见已经有ans值的点，就不用再继续滑下去，而是直接将这个ans值返回给上一级。

然后，对每个未滑行到的点来说，初始滑行距离step是1，遍历上下左右四个可能的点，更新step

`if(nums[next] < nums[now])       step = max(step, dfs(next)+1);`

然后在返回前将ans[now]的值设置为step。

最后遍历ans矩阵，得到最大的ans值即可。

```c++
#include<bits/stdc++.h>

using namespace std;

int r, c, nums[101][101], ans[101][101];
int dire[4][2]={{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

int dfs(int x, int y){
    if(ans[x][y] != 0){
        return ans[x][y];
    }
    int step = 1;
    for(int i = 0; i < 4; i++){
        int xx = x + dire[i][0], yy = y + dire[i][1];
        if(xx < 0 || xx >= r || yy < 0 || yy >= c)
            continue;
        if(nums[xx][yy] < nums[x][y])
            step = max(step, dfs(xx, yy)+1);
    }
    ans[x][y] = step;
    return step;
}
int main(){
    scanf("%d%d", &r, &c);
    for(int i = 0; i < r; i++){
        for(int j = 0; j < c; j++)
            scanf("%d", &nums[i][j]);
    }
    int res = 0;
    dfs(0, 0);
    for(int i = 0; i < r; i++){
        for(int j = 0; j < c; j++)
            res = max(res, dfs(i, j));
    }
    cout<<res<<endl;
    return 0;
}

```

### 3. P1433 吃奶酪

#### 思路：

求走过各个点的最短路径，n=15，最朴素的思路是DFS暴搜。。只能拿90分。最后一个case超时了。

这里需要从原点出发，有一个小技巧是把原点也作为其中一个点，然后从这个点出发去暴搜。

另一个技巧是预处理每两个点之间的距离存到数组当中去。

另外就是如果当前路径的长度已经超过了ans就直接返回（剪枝）。

#### DFS

```c++
#include<bits/stdc++.h>

using namespace std;
int n;
double x, y;
vector<pair<double, double> > pos;
double dis[16][16]; //dis[i][j] = distance between pos[i] and pos[j]
int used[16], cnt = 0;
double ans = 1000000000.0;
void getDis(int n){
    for(int i = 0; i <= n; i++){
        double x1 = pos[i].first, y1 = pos[i].second;
        for(int j = i+1; j <= n; j++){
            double x2 = pos[j].first, y2 = pos[j].second;
            dis[i][j] = sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2));
            dis[j][i] = dis[i][j];
        }
    }
}


void dfs(int index, double len){
    if(len > ans)
        return;
    if(cnt == n){
        ans = min(ans, len);
        return;
    }
    used[index] = 1;
    for(int i = 1; i <= n; i++){
        if(used[i] == 0){
            cnt++;
            dfs(i, len+dis[index][i]);
            cnt--;
        }
    }
    used[index] = 0;
    return;
}
int main(){
    scanf("%d", &n);
    pos.push_back(make_pair(0, 0));
    for(int i = 1; i <= n; i++){
        scanf("%lf%lf", &x, &y);
        pos.push_back(make_pair(x, y));
    }
    getDis(n);
    dfs(0, 0);
    printf("%.2f", ans);
    return 0;
}
```

DFS是不行滴，即使剪枝了也会超时。那就只能学一下别人的状态压缩DP是怎么做的咯。

DP状态压缩的大概意思是将某个状态（集合）用位运算表示，比如在这里，可以用`f[i][S]`表示从第i个点出发（i必须在S里）遍历集合S中的点的最短路径。（集合S可以用二进制的1和0代表第几个点是否在该集合中）。由此，f的数组就必须开成f[n]\[(1<<n) - 1]这么大。

然后在DP的时候，需要遍历所有的状态（S从1到(1<<n)-1），对每个状态，遍历每个点，首先确保该点在该状态的集合中，然后再遍历每个点，找到从该点出发的遍历整个集合（(1<<n)-1）的最短路径。

dp状态转移方程：

`f[j][s] = min(f[j][s], f[k][s - (1 << (j-1))] + dis[j][k]);`

对于从第j个点出发遍历集合S的最短路径，等于从第k个点出发遍历除j以外的点的最短路径+j和k之间的距离取最小值。

遍历完成以后，`f[1][(1<<n)-1]到f[n][(1<<n)-1]`就是从每个点出发遍历所有点的最短路径，再加上从原点(0, 0)出发的距离，取最小值输出就是想要的答案（保留两位小数）。

#### 状态压缩DP

```c++
#include<bits/stdc++.h>

using namespace std;
int n;
double x, y;
vector<pair<double, double> > pos;
double dis[16][16]; //dis[i][j] = distance between pos[i] and pos[j]
double ans = 1000000000.0;
double f[16][(1<<16)];// f[i][S] means the minimum distance started from position
                      // i and go over all points of set S.(if S=1111 means point 1-4).

void getDis(int n){
    for(int i = 0; i <= n; i++){
        double x1 = pos[i].first, y1 = pos[i].second;
        for(int j = i+1; j <= n; j++){
            double x2 = pos[j].first, y2 = pos[j].second;
            dis[i][j] = sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2));
            dis[j][i] = dis[i][j];
        }
    }
}

int main(){
    scanf("%d", &n);
    pos.push_back(make_pair(0, 0));
    for(int i = 1; i <= n; i++){
        scanf("%lf%lf", &x, &y);
        pos.push_back(make_pair(x, y));
    }
    getDis(n);
    memset(f,127,sizeof(f));
    for(int s = 1; s <= (1 << n)-1; s++){
        for(int j = 1; j <= n; j++){
            if((s & (1 << (j-1))) == 0)//means j-th point is not in set S;
                continue;
            if(s == (1 << (j-1))){
                f[j][s] = 0; //the set has only one point.
                continue;
            }
            for(int k = 1; k <= n; k++){
                if(s & (1 << (k-1)) == 0 || j == k)
                    continue;
                f[j][s] = min(f[j][s], f[k][s - (1 << (j-1))] + dis[j][k]);
            }

        }
    }
    ans = -1;
    for(int i = 1; i <= n; i++){
        if(ans == -1.0)
            ans = f[i][(1<<n) - 1] + dis[0][i];
        else
            ans = min(ans, f[i][(1<<n) - 1] + dis[0][i]);
    }
    printf("%.2f", ans);
    return 0;
}

```

### 4. P1074 靶形数独

#### 思路：

这道题的意思是在所有可能的数独的解中，找到一个符合题意的能得到最大分数的值。

首先最简单的，我们可以使用三个数组row, col, box，row[num]\[i]表示数字num在第i行是否已经出现了，col，box同理。box表示的是数独的第几个九宫格。这三个数组在输入数独的时候就已经填写完成了。

搜索是肯定要搜索，但是怎么剪枝才是关键。

看了评论区的意思就是，一行一行来，把0个数最小的行优先搜索。

这样就需要在输入完以后进行**预处理**，统计一下每一行没有填的位置的个数，然后把9行按照0的个数排序，再然后，根据这个顺序将每个需要填数字的位置放进一个数组里(`search_points`)。

做完这一部分工作以后，就到了紧张刺激（按套路）的DFS环节啦。

DFS的时候首先判断一下是不是所有的点都填完了，如果是，就更新一下最高分的值；否则，就将当前index对应的数独位置取出，得到对应的行列。

注意：这里因为我在预处理的时候放进`search_points`数组里的值(`x*9+y`)对应的数独坐标本身就是没有填充过数的位置坐标，所以其实不需要判断一下该位置有没有数字。

得到对应位置后，遍历1到9这几个数字，看看这几个数字有没有能填进该位置的，能填就填然后`dfs(index+1)`。`dfs(index+1)`之前需要修改状态，dfs完了以后需要回溯。

总而言之，思路是比较清晰的。

再注意：题目说数独无解就输出-1，这里初始化的时候将最高分初始化为-1了，不用多开一个变量。

#### 剪枝技巧：

DFS的时候尽量从分叉少的地方开始搜索。

#### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
int nums[9][9], row[10][9], col[10][9], box[10][9];
// row[i][j] means for number i, the j-th row can be filled or not(1).
int score[9][9] = {
    {6, 6, 6, 6, 6, 6, 6, 6, 6},
    {6, 7, 7, 7, 7, 7, 7, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 9, 10, 9, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 7, 7, 7, 7, 7, 7, 6},
    {6, 6, 6, 6, 6, 6, 6, 6, 6},
};
vector<pair<int, int> > row_zeros;
int search_points[81];

int most = -1;

bool cmp(pair<int, int> a, pair<int, int> b){
    return a.second == b.second ? a.first < b.first : a.second < b.second;
}
int getBox(int x, int y){
    return (x/3)*3 + y / 3;
}

void print(int a[10][9]){
    for(int i = 1; i <= 9; i++){
        for(int j = 0; j < 9; j++){
            cout<<a[i][j]<<" ";
        }
        cout<<endl;
    }

}

int getScore(){
    int res = 0;
    for(int i = 0; i < 9; i++)
        for(int j = 0; j < 9; j++)
            res += score[i][j] * nums[i][j];
    return res;
}

void dfs(int index){
    if(search_points[index] == -1){
        most = max(most, getScore());
        return;
    }
    int value = search_points[index];
    int x = value / 9, y = value % 9;
//    if(nums[x][y] != 0)
//        dfs(index + 1);
    for(int i = 1; i < 10; i++){
        int b = getBox(x, y);
        if(!row[i][x] && !col[i][y] && !box[i][b]){
            nums[x][y] = i;
            row[i][x] = 1, col[i][y] = 1, box[i][b] = 1;
            dfs(index + 1);
            row[i][x] = 0, col[i][y] = 0, box[i][b] = 0;
            nums[x][y] = 0;
        }
    }
}

void init(){
    // count how many zeros in each row;
    for(int i = 0; i < 9; i++){
        int cnt = 0;
        for(int j = 0; j < 9; j++)
            if(nums[i][j] == 0)
                cnt++;
        row_zeros.push_back(make_pair(i, cnt));
    }
    sort(row_zeros.begin(), row_zeros.end(), cmp);
    // put the points to be filled into array in ascending order of
    // the zero numbers of its row;
    int index = 0;
    memset(search_points, -1, sizeof(search_points));
    for(int i = 0; i < 9; i++){
        int row = row_zeros[i].first;
        for(int j = 0; j < 9; j++){
            if(nums[row][j] == 0){
                search_points[index++] = row * 9 + j;
            }
        }
    }

}
int main(){
    for(int i = 0; i < 9; i++){
        for(int j = 0; j < 9; j++){
            scanf("%d", &nums[i][j]);
            int num = nums[i][j];
            row[num][i] = 1;
            col[num][j] = 1;
            box[num][getBox(i, j)] = 1;

        }
    }
    init();
    dfs(0);
    cout<<most<<endl;
    return 0;
}
```



