---
title: leetcode1
date: 2020-03-29 23:08:30
tags: [leetcode]
series: [blog]
featured: true
toc: true
comments: true
---

Table of Contents
=================

* [820. 单词的压缩编码](#820-单词的压缩编码)
    * [思路1－字典树：](#思路1字典树)
    * [思路2:](#思路2)
* [208. 实现 Trie (前缀树)](#208-实现-trie-前缀树)
    * [思路：](#思路)
* [1162. 地图分析](#1162-地图分析)
	* [思路：](#思路-1)
	* [多源Dijkstra](#多源dijkstra)
	* [多源BFS](#多源bfs)
* [5370. 设计地铁系统](#5370-设计地铁系统)
	* [思路：](#思路-2)
* [5369. 统计作战单位数](#5369-统计作战单位数)
	* [思路:](#思路-3)
* [5368. 找出幸运数](#5368-找出幸运数)
	* [思路：](#思路-4)



### 820. 单词的压缩编码

给定一个单词列表，将这个列表编码成一个索引字符串S与一个索引列表A，求最短的编码长度。

#### 思路1－字典树：

```c++
struct Node{
    Node *next[26];
    bool flag;
    Node(){
        for(int i = 0; i < 26; i++)
            next[i] = NULL;
        flag = false;
    }
};
void Insert(string str, int pos, Node* root){
    if(pos < 0){
        root->flag = true;
        return;
    }
    int ind = str[pos] - 'a';
    if(root->next[ind] == NULL)
        root->next[ind] = new Node();
    Insert(str, pos-1, root->next[ind]);
}

int Count(Node* root, int cnt){
    int leaf = 1, sum = 0;
    for(int i = 0; i < 26; i++){
        if(root->next[i] != NULL){
            leaf = 0;
            sum += Count(root->next[i], cnt+1);
        }
    }
    if(leaf)
        return cnt+1;
    return sum;
}
```

最后解答：

```c++
class Solution {
public:
    struct Node{
        Node *next[26];
        bool flag;
        Node(){
            for(int i = 0; i < 26; i++)
                next[i] = NULL;
            flag = false;
        }
    };
    void Insert(string str, int pos, Node* root){
        if(pos < 0){
            root->flag = true;
            return;
        }
        int ind = str[pos] - 'a';
        if(root->next[ind] == NULL)
            root->next[ind] = new Node();
        Insert(str, pos-1, root->next[ind]);
    }

    int Count(Node* root, int cnt){
        int leaf = 1, sum = 0;
        for(int i = 0; i < 26; i++){
            if(root->next[i] != NULL){
                leaf = 0;
                sum += Count(root->next[i], cnt+1);
            }
        }
        if(leaf)
            return cnt+1;
        return sum;
    }
    int minimumLengthEncoding(vector<string>& words) {
        Node* root = new Node();
        for(int i=0; i < words.size(); i++){
            Insert(words[i], words[i].size()-1, root);
        }
        return Count(root, 0);
    }
};
```

#### 思路2:

将每个字符串反转，然后将字符串排序，只需要检查前一个字符串是否是后一个字符串的子川，是就丢弃前一个字符串；

```c++
class Solution {
public:
    int minimumLengthEncoding(vector<string>& words) {
        int n = words.size();
        vector<string> rs;
        for(int i = 0; i < n; i++){
            reverse(words[i].begin(), words[i].end());
            rs.push_back(words[i]);
        }
        sort(rs.begin(), rs.end());
        int cnt = 0;
        for(int j = 0; j < n-1; j++){
            if(rs[j+1].find(rs[j]) == 0)
                continue;
            cnt += rs[j].size() + 1;
        }
        cnt += rs[n-1].size() + 1;
        return cnt;
    }

};
```

### 208. 实现 Trie (前缀树)

#### 思路：

这道题与前一题类似（就是为了巩固前缀树的知识才写的）。

```c++
class Trie {
public:
    /** Initialize your data structure here. */
    struct Node{
        bool flag;
        Node* next[26];
        Node(){
            for(int i = 0; i < 26; i++)
                next[i] = NULL;
            flag = false;
        }
    };
    Node* root;
    Trie() {
        root = new Node();
    }
    
    /** Inserts a word into the trie. */
    void insert(string word) {
        Node* cur = root;
        for(int i = 0; i < word.size(); i++){
            if(cur->next[word[i]-'a'] == NULL)
                cur->next[word[i]-'a'] = new Node();
            cur = cur->next[word[i]-'a'];
        }
        cur->flag = true;
    }
    
    /** Returns if the word is in the trie. */
    bool search(string word) {
        Node* cur = root;
        for(int i = 0; i < word.size(); i++){
            if(cur->next[word[i]-'a'] == NULL)
                return false;
            cur = cur->next[word[i]-'a'];
        }
        return cur->flag;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    bool startsWith(string prefix) {
        Node* cur = root;
        for(int i = 0; i < prefix.size(); i++){
            if(cur->next[prefix[i]-'a'] == NULL)
                return false;
            cur = cur->next[prefix[i]-'a'];
        }
        return true;
    }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 */
```

### 1162. 地图分析

#### 思路：

要找出离陆地区域最远的海洋区域，这道题有4种做法，详见[官方题解](https://leetcode-cn.com/problems/as-far-from-land-as-possible/solution/di-tu-fen-xi-by-leetcode-solution/)

这里只写了多源Dijkstra和多源BFS两种：

#### 多源Dijkstra

```c++
static constexpr int INF = int(1E6);
static constexpr int dire[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

int dis[110][110];
int Dijkstra(vector<vector<int>>& grid){
    struct Node{
        int v, x, y;
        bool operator < (const Node& test) const{
            return v > test.v;
        }
    };
    priority_queue<Node> q;
    int n = grid.size();
    //initialization
    for(int i = 0; i < n; i++){
        for(int j = 0; j < n; j++){
            if(grid[i][j]){
                dis[i][j] = 0;
                q.push({0, i, j});
            }else{
                dis[i][j] = INF;
            }
        }
    }
    while(!q.empty()){
        auto top = q.top();
        int value = top.v, x = top.x, y = top.y;
        q.pop();
        for(int i = 0; i < 4; i++){
            int xx = x+dire[i][0], yy = y+dire[i][1];
            if(xx < 0 || xx >= n || yy < 0 || yy >= n)
                continue;
            if(value+1 < dis[xx][yy]){
                dis[xx][yy] = value + 1;
                q.push({dis[xx][yy], xx, yy});
            }
        }
    }
    int ans = -1;
    for(int i = 0; i < n; i++){
        for(int j = 0; j < n; j++){
            if(!grid[i][j])
                ans = max(ans, dis[i][j]);
        }
    }
    return (ans == INF)?-1:ans;
}
```

#### 多源BFS

```c++
static constexpr int INF = int(1E6);
static constexpr int dire[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

int dis[110][110];
int bfs(vector<vector<int>>& grid){
    queue<pair<int, int> > q;
    int n = grid.size();
    for(int i = 0; i < n; i++)
        for(int j = 0; j < n; j++){
            if(grid[i][j]){
                dis[i][j] = 0;
                q.push(make_pair(i,j));
            }
            else
                dis[i][j] = INF;
        }
    while(!q.empty()){
        int x = q.front().first, y = q.front().second;
        q.pop();
        for(int i = 0; i < 4; i++){
            int xx = x + dire[i][0], yy = y + dire[i][1];
            if(xx < 0 || xx >= n || yy < 0 || yy >= n)
                continue;
            if(dis[xx][yy] > dis[x][y] + 1){
                dis[xx][yy] = dis[x][y] + 1;
                q.push(make_pair(xx, yy));
            }
        }
    }
    int ans = -1;
    for(int i = 0; i < n; i++){
        for(int j = 0; j < n; j++){
            if(!grid[i][j])
                ans = max(ans, dis[i][j]);
        }
    }
    return (ans == INF)?-1:ans;
}
```

观察代码你会发现，Dijkstra和BFS具有高度相似性。其实本质上只是前者将后者的队列换成了优先队列而已。

至于多源最短路问题和单源最短路问题相比，无非就是建立一个超级源头，把所有的源头都作为它的连通点并且权值为0。

> 　思考：海洋区域和陆地区域，应该哪一个作为源点集？ 也许你分析出「我们需要找一个海洋区域，满足它到陆地的最小距离是最大」会把海洋区域作为源点集。我们可以考虑后续的实现，我们知道 Dijkstra 中一个 d 数组用来维护当前源点集到其他点的最短路，而对于源点集中的任意一个点 ss，d[s_x][s_y] = 0，这很好理解，源点到源点的最短路就是 0。如果我们把海洋区域作为源点集、陆地区域作为目标点集，假设 tt 是目标点集中的一个点，算法执行结束后 d[t_x][t_y] 就是海洋区域中的点到 tt 的最短距离，但是我们却不知道哪些 tt 是海洋区域的这些点的「最近陆地区域」，我们也不知道每个 ss 距离它的「最近陆地区域」的曼哈顿距离。考虑我们把陆地区域作为源点集、海洋区域作为目标点集，目标点集中的点 tt 对应的 d[t_x][t_y] 就是海洋区域 tt 对应的距离它的「最近陆地区域」的曼哈顿距离，正是我们需要的，所以应该把陆地区域作为源点集。
>
> 最终我们只需要比出 d[t_x][t_y] 的最大值即可。Dijkstra 算法在初始化 d 数组的时候，把每个元素预置为 INF，所以如果发现最终比出的最大值为 INF，那么就返回 -1。
>
> 由于这里的边权为 1，也可以直接使用多源 BFS，在这里每个点都只会被松弛一次。
>
> －－－－leetcode官方解答

更多解答请参考官网。

### 5370. 设计地铁系统

#### 思路：

利用STL缓存信息。

```c++
class UndergroundSystem {
public:
    // startStation endStation, time, cnt
    map<pair<string, string>, pair<int, int>> record;
    // user id, startStation, startTime;
    map<int, pair<string, int> > running;
    UndergroundSystem() {
        record.clear();
        running.clear();
    }
    
    void checkIn(int id, string stationName, int t) {
        running[id] = make_pair(stationName, t);
    }
    
    void checkOut(int id, string stationName, int t) {
        auto it = running.find(id);
        record[make_pair(it->second.first, stationName)].first += (t - it->second.second);
        record[make_pair(it->second.first, stationName)].second++; 
        running.erase(it);
    }
    
    double getAverageTime(string startStation, string endStation) {
        double time = (double) record[make_pair(startStation, endStation)].first;
        int cnt = record[make_pair(startStation, endStation)].second;
        return time / cnt;
    }
};

/**
 * Your UndergroundSystem object will be instantiated and called as such:
 * UndergroundSystem* obj = new UndergroundSystem();
 * obj->checkIn(id,stationName,t);
 * obj->checkOut(id,stationName,t);
 * double param_3 = obj->getAverageTime(startStation,endStation);
 */
```

### 5369. 统计作战单位数

#### 思路:

这道题是想求出数组中顺序三元组的个数和逆序三元组的个数；直接暴力。

```c++
class Solution {
public:
    int numTeams(vector<int>& rating) {
        int cnt = 0;
        for(int i = 0; i < rating.size(); i++){
            for(int j = i+1; j < rating.size(); j++){
                if(rating[j] > rating[i]){
                    for(int k = j+1; k < rating.size();k++)
                        if(rating[k] > rating[j])
                            cnt++;
                }
            }
        }
        for(int i = 0; i < rating.size(); i++){
            for(int j = i+1; j < rating.size(); j++){
                if(rating[j] < rating[i]){
                    for(int k = j+1; k < rating.size();k++)
                        if(rating[k] < rating[j])
                            cnt++;
                }
            }
        }
        return cnt;
    }
};
```

### 5368. 找出幸运数

#### 思路：

这道题是找出符合要求的数字。哈希即可。**注意局部变量的初始化。**

```c++
class Solution {
public:
    int findLucky(vector<int>& arr) {
        int cnt[501];
        memset(cnt, 0, sizeof(cnt));
        for(int i = 0; i < arr.size(); i++)
            cnt[arr[i]]++;
        for(int i = 500; i >= 1; i--){
            if(cnt[i] == i)
                return i;
        }
        return -1;
    }
};
```

