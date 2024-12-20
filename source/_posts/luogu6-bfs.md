---
title: 洛谷6-广度优先搜索
date: 2020-02-05 00:53:52
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1162 填涂颜色](#1-p1162-填涂颜色)
	* [思路：](#思路)
	* [DFS](#dfs)
	* [BFS](#bfs)
* [2. P1032 字串变换](#2-p1032-字串变换)
	* [思路：](#思路-1)
	* [双向BFS：](#双向bfs)
	* [注意：](#注意)
* [3. P1141 01迷宫](#3-p1141-01迷宫)
	* [思路：](#思路-2)
	* [DFS](#dfs-1)
	* [BFS](#bfs-1)
* [4. P1443 马的遍历](#4-p1443-马的遍历)
	* [思路：](#思路-3)
	* [代码：](#代码)



### 1. P1162 填涂颜色

#### 思路：

这道题是经典的`BFS`题目，用`DFS`也能写，但是为了锻炼`BFS`我还是坚持用`BFS`。为了找出最大的被1包围的联通块，我们需要遍历每个点，用vis数组标记访问。最后没有被访问到的点就是被1包围的点，因为1的位置被限制为访问过，不会继续探索了。但这里会有一个问题，如果是边角位置的被1包围的0（即1将整个map分割成多个孤岛）我们只要中间这个完整的。可以将整个map外层裹上一层0，这样就确保了被1隔离在外面的区域一定是由0连在一起的，就能通过0访问到，而只有被1包围在里面的联通块不会被访问到。最后代码如下：

#### DFS

```c++
#include<bits/stdc++.h>

using namespace std;
int n, nums[32][32];
int vis[32][32];
int dire[4][2] = { {-1, 0}, {1, 0}, {0, 1}, {0, -1}};
void dfs(int x, int y){
    if(vis[x][y])
        return;
    vis[x][y] = 1;
    for(int i = 0; i < 4; i++){
        int xxx = x + dire[i][0], yyy = y + dire[i][1];
        if(xxx < 0 || yyy < 0 || xxx >= n + 2 || yyy >= n + 2)
            continue;
        dfs(xxx, yyy);
    }
}

int main(){
    scanf("%d", &n);
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= n; j++){
            scanf("%d", &nums[i][j]);
            if(nums[i][j] == 1)
                vis[i][j] = 1;
        }
    }
    dfs(0, 0);
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= n; j++){
            if(vis[i][j] == 1)
                cout<<nums[i][j]<<" ";
            else
                cout<<2<<" ";
        }
        cout<<endl;
    }
    return 0;
}

```

#### BFS

```c++
#include<bits/stdc++.h>

using namespace std;
int n, nums[32][32];
int vis[32][32];
queue<pair<int, int> > q;
int dire[4][2] = { {-1, 0}, {1, 0}, {0, 1}, {0, -1}};
void bfs(int x, int y){
    vis[x][y] = 1;
    q.push(make_pair(x, y));
    while(!q.empty()){
        int xx = q.front().first, yy = q.front().second;
        q.pop();
        for(int i = 0; i < 4; i++){
            int xxx = xx + dire[i][0], yyy = yy + dire[i][1];
            if(xxx < 0 || yyy < 0 || xxx >= n + 2 || yyy >= n + 2)
                continue;
            if(vis[xxx][yyy] == 1)
                continue;
            vis[xxx][yyy] = 1;
            q.push(make_pair(xxx, yyy));

        }
    }
}

int main(){
    scanf("%d", &n);
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= n; j++){
            scanf("%d", &nums[i][j]);
            if(nums[i][j] == 1)
                vis[i][j] = 1;
        }
    }
    bfs(0, 0);
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= n; j++){
            if(vis[i][j] == 1)
                cout<<nums[i][j]<<" ";
            else
                cout<<2<<" ";
        }
        cout<<endl;
    }
    return 0;
}
```

### 2. P1032 字串变换

#### 思路：

常规做法是BFS搜索每个可能的解答，这里学到了一种新的做法，双向BFS。

#### 双向BFS：

适用于**知道搜索起点状态和终点状态**，需要找到最短的搜索路径的情况。如果要求字典序的最短路径则不行，因为从后向前搜索的时候没办法保证字典序，只能保证最短路径。

双向BFS的时候，依次交替向前和向后迭代。当向前搜索和向后搜索的某个位置出现交叉时，此时必定是最短路径，返回结果即可。

**双向bfs关键就是要记录两棵搜索树的状态，通过状态判断是否出现相遇**

具体还需要通过不同的题目进行练习。

代码：

```c++
#include<bits/stdc++.h>

using namespace std;
int n=1;
string a, b, ta[10], tb[10];
map <string, int> A, B; // string, step
queue<string> fd, bd;  // forward, backward;
int step, ans;

int bfs(string up, string down){
    fd.push(up);
    A[up] = step;
    bd.push(down);
    B[down] = step;
    string s, s2;
    while(++step < 6){
        while(A[fd.front()] == step-1){
            s = fd.front();
            fd.pop();
            for(int i = 1; i <= n; i++){
                int index = 0;
                while(index < s.size()){
                    if(s.find(ta[i], index) == s.npos)
                        break;
                    string s2 = s;
                    s2.replace(s2.find(ta[i], index), ta[i].size(), tb[i]);
                    if(A.find(s2) != A.end()){
                        index++;
                        continue;
                    }
                    if(B.find(s2) != B.end())
                        return step * 2 - 1;
                    fd.push(s2);
                    A[s2] = step;
                    index++;
                }
            }
        }
        while(B[bd.front()] == step-1){
            s = bd.front();
//            cout<<y<<endl;
            bd.pop();
            for(int i = 1; i <= n; i++){
                int index = 0;
                while(index < s.size()){
                    if(s.find(tb[i], index) == s.npos)
                        break;
                    string s2 = s;
                    s2.replace(s2.find(tb[i], index), tb[i].size(), ta[i]);
                    if(B.find(s2) != B.end()){
                        index++;
                        continue;
                    }
                    if(A.find(s2) != A.end())
                        return step * 2;
                    bd.push(s2);
                    B[s2] = step;
                    index++;
                }
            }
        }
    }
    return -1;
}
int main(){
//    freopen("in.txt","r",stdin);
//    freopen("out.txt","w",stdout);
    cin>>a>>b;
    while(cin>>ta[n]>>tb[n]){
        n++;
    }
    ans = bfs(a, b);
    if(ans == -1)
        cout<<"NO ANSWER!"<<endl;
    else cout<<ans<<endl;

    return 0;
}

```

#### 注意：

经常会碰到无限制输入行数的情况，此时需要使用`while(cin>>a)`的句式，但是在本地编译是console是无法结束输入的，这个时候需要使用上文中注释掉的`freopen`进行本地调试，在同级目录下创建对应的`in.txt, out.txt`文件，将输入复制到`in.txt`文件中，然后编译运行，在`out.txt`文件中查看结果即可。

### 3. P1141 01迷宫

#### 思路：

这道题非常有意思，如果使用经典dfs，很容易拿到70分。剩下三个case会`TLE`。原因很简单就是输入样例太多。所以需要想办法保存已经搜索过的结果。最开始参考了一些题解（并查集之类的），但是还是想试试宽搜能不能做到这一点。这里有一个技巧，就是所有能跳到的位置的答案都是一样的（处于同一个联通块的答案是一样的）。因此可以设置一个flag值，表明该点属于哪一个联通块；在每次输入查询点的时候，首先看看这个查询点的flag有没有被设置为相应的联通块号码，如果有，直接输出那个联通块的答案就可以了。没有的话就进行宽搜。在宽搜的时候，我们应该想到，对于一个符合要求的点（可以跳到的位置），如果这个点已经有flag值（已经属于某个联通块），那么你马上就可以停止搜索并返回了。从main函数里每次调用`dfs`的时候，就是出现了新的未在当前所有联通块里的点，这时候就应该使联通块号d++，然后对当前点进行宽搜。而**宽搜的时候实际上会将属于这个联通块的所有的点都访问一遍，这个时候直接在访问的过程中设置每一个点的flag值为当前联通块号，并且将对应联通块的答案+1。**

但是实际操作的时候，还是碰见了超时，因为我单独开了一个visited数组，每次输入的时候调用了`memset`函数将visited清零，这个操作太耗时间了。并且，flag数组完全可以替代visited的功能，所以又修正了一下。

最后代码如下：

#### DFS

```c++
#include<bits/stdc++.h>

using namespace std;
int n, m;
char nums[1010][1010];
int x, y, flag[1010][1010];
int dire[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
int ans[1000010], d = 1;

void dfs(int x, int y){
    if(flag[x][y] != 0)
        return;
    ans[d] += 1;
    flag[x][y] = d;
    for(int i = 0; i < 4; i++){
        int xxx = x + dire[i][0], yyy = y + dire[i][1];
        if(xxx < 0 || yyy < 0 || xxx >= n || yyy >= n)
            continue;
        if(nums[x][y] != nums[xxx][yyy]){
            dfs(xxx, yyy);
        }
    }

}
int main(){
//    freopen("in.txt", "r", stdin);
//    freopen("out.txt", "w", stdout);
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++)
        scanf("%s", &nums[i]);
    for(int j = 0; j < m; j++){
        scanf("%d%d", &x, &y);
        if(flag[x-1][y-1] == 0){
            d++;
            dfs(x-1, y-1);
        }
        cout<<ans[flag[x-1][y-1]]<<endl;
    }
    return 0;
}
```

我最开始是想用`BFS`的，然后才发现自己写的根本不是`BFS`而是带队列的`DFS`（傻傻分不清）。。。惭愧惭愧。。现在改了一个`BFS`的版本补充在下方。顺带把上面的`DFS`版本改了一下（`DFS`要个锤子的队列。。。）

#### BFS

```c++
#include<bits/stdc++.h>

using namespace std;
int n, m;
char nums[1010][1010];
int x, y, flag[1010][1010];
int dire[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
queue<pair<int, int> > q;
int ans[1000010], d = 1;

void bfs(int x, int y){
    ans[d] = 1;
    flag[x][y] = d;
    q.push(make_pair(x, y));
    while(!q.empty()){
        int xx = q.front().first, yy = q.front().second;
        q.pop();
        for(int i = 0; i < 4; i++){
            int xxx = xx + dire[i][0], yyy = yy + dire[i][1];
            if(xxx < 0 || yyy < 0 || xxx >= n || yyy >= n)
                continue;
            if(nums[xx][yy] != nums[xxx][yyy] && flag[xxx][yyy] == 0){
                q.push(make_pair(xxx, yyy));
                ans[d] += 1;
                flag[xxx][yyy] = d;
            }
        }
    }
}
int main(){
//    freopen("in.txt", "r", stdin);
//    freopen("out.txt", "w", stdout);
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++)
        scanf("%s", &nums[i]);
    for(int j = 0; j < m; j++){
        scanf("%d%d", &x, &y);
        if(flag[x-1][y-1] == 0){
            d++;
            bfs(x-1, y-1);
        }
        cout<<ans[flag[x-1][y-1]]<<endl;
    }
    return 0;
}

```

### 4. P1443 马的遍历

#### 思路：

这道题也是经典的`BFS`题目，这道题让我意识到了`BFS`和`DFS`两者在写法上的区别（`BFS`不能递归啊××）。然后卡了的地方就是宽5格对齐，这里要使用`printf`的格式化输出。`printf("%-5d", &num);`

#### 代码：

```c++
#include<bits/stdc++.h>

using namespace std;
int n, m, x, y;
int dire[8][2] = {{1, 2}, {1, -2}, {2, 1}, {2, -1}, {-1, 2}, {-1, -2}, {-2, -1}, {-2, 1}};
int vis[401][401], step, ans[401][401];
queue<pair<int, int> > q;

void bfs(int x, int y){
    q.push(make_pair(x, y));
    vis[x][y] = 1;
    while(!q.empty()){
        int xx = q.front().first, yy = q.front().second;
        q.pop();
        for(int i = 0; i < 8; i++){
            int xxx = xx + dire[i][0], yyy = yy + dire[i][1];
            if(xxx < 0 || xxx >= n || yyy < 0 || yyy >= m)
                continue;
            if(vis[xxx][yyy] != 0){
                ans[xxx][yyy] = min(ans[xxx][yyy], ans[xx][yy] + 1);
                continue;
            }
            ans[xxx][yyy] = ans[xx][yy] + 1;
            q.push(make_pair(xxx, yyy));
            vis[xxx][yyy] = 1;
        }
    }
}
int main(){
    scanf("%d%d%d%d", &n, &m, &x, &y);
    memset(ans, -1, sizeof(ans));
    ans[x-1][y-1] = 0;
    bfs(x-1, y-1);
    for(int i = 0; i < n; i++){

        for(int j = 0; j < m; j++){
            printf("%-5d", ans[i][j]);
        }
        cout<<endl;
    }
    return 0;
}
```



