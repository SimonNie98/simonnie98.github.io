---
title: 洛谷4-深度优先搜索(2)
date: 2020-01-21 00:21:47
tags: [洛谷]
categories: [Algorithms]
toc: true
comments: true
---
<!-- more -->
Table of Contents
=================

* [1. P1019 单词接龙](#1-p1019-单词接龙)
	* [思路：](#思路)
* [2. P1101 单词方阵](#2-p1101-单词方阵)
	* [思路：](#思路-1)
	* [注意：](#注意)



### 1. P1019 单词接龙

#### 思路：

这道题是先输入一个n，然后输入n行，每行是一个单词，最后输入一个单字母(第n+1)，然后进行拼接。

需要写的函数有：判断两个字符串是否可以拼接，dfs暴力搜索；

```c++
#include<bits/stdc++.h>

using namespace std;
int n;
vector<string> words;
int visited[110], ans;
string tmp;
//
int isOk(string a, string b){
    if(a == words[n])
        return a[0] == b[0];
    // return the min value of repeated string length.
    for(int i = 1; i < min(a.size(), b.size()); i++){
        int flag = 1;
        for(int j = a.size() - i, k = 0; j < a.size() && k < i; j++, k++)
            if(a[j] != b[k])
                flag = 0;
        if(flag)
            return i;
    }
    return 0;
}

string add_str(string a, string b, int len){
    for(int i = len; i < b.size(); i++){
        a += b[i];
    }
    return a;
}

string sub_str(string a, int len){
    string ans = "";
    for(int i = 0; i < len; i++)
        ans += a[i];
    return ans;
}

void dfs(string str, int len){
    for(int i = 0; i < n; i++){
        if(visited[i] >= 2) continue;
        else{
            int a = 0;
            a = isOk(str, words[i]);
            if(str != "" && !a)
                continue;
            str = add_str(str, words[i], a);
            visited[i]++;
//            cout<<"/:"<<str<<endl;
            dfs(str, len + words[i].size() - a);
            str = sub_str(str, len);
            visited[i]--;
        }
    }
    ans = max(ans, len);
}

int main(){
    scanf("%d", &n);
    for(int i = 0; i <= n; i++){
        cin>>tmp;
        words.push_back(tmp);
    }
    dfs(words[n], 1);
    cout<<ans<<endl;
    return 0;
}

```

一遍AC但是花了很久的时间。

### 2. P1101 单词方阵

#### 思路：

单词方阵需要对每个位置判断是不是target字符串的起始字符，如果是，对该位置进行八个方向的扫描，判断其是否能够得到完整的target字符串。如果可以，就将对应位置标记为1并进行染色，否则就标记为0。最后对染色部分进行处理，输出。即可。

一遍AC但是我没用dfs。

#### 注意：

1. 当发现该位置存在某个方位可以得到target串之后，别忘了将当前位置也染色。
2. 获取输入时，如果使用`scanf("%c")`会从缓冲区读入`\n`，不如使用`cin>>`接收一行作为输入。

```c++
#include<bits/stdc++.h>

using namespace std;

int n;
string s[101], tmp;
int flag[101][101];
string target = "yizhong";
int dire[8][2] = {{1, 0}, {1, 1}, {0, 1}, {-1, 1}, {-1, 0}, {-1, -1}, {0, -1}, {1, -1}};

void search(){
    for(int i = 0; i < n; i++)
    for(int j = 0; j < n; j++){
        if(s[i][j] != target[0])
            continue;
        else
            for(int k = 0; k < 8; k++){
                int xx = i + dire[k][0], yy = j + dire[k][1];
                if(xx < 0 || yy < 0 || xx >= n || yy >= n)
                    continue;
                int t = 1, index = 1;
                while(index < target.size()){
                    if(xx < 0 || yy < 0 || xx >= n || yy >= n || s[xx][yy] != target[index]){
                        t = 0;
                        break;
                    }
                    xx += dire[k][0], yy += dire[k][1];
                    index++;
                }
                if(t){
                    index = 1, xx = i + dire[k][0], yy = j + dire[k][1];
                    while(index < target.size()){
                        flag[xx][yy] = 1;
                        xx += dire[k][0], yy += dire[k][1];
                        index++;
                    }
                    flag[i][j] = 1;
                }
            }
    }
}

void draw(){
    for(int i = 0; i < n; i++)
    for(int j = 0; j < n; j++){
        if(flag[i][j] == 1)
            continue;
        else s[i][j]='*';
    }
}

void print(){
    for(int i = 0; i < n; i++){
        for(int j = 0; j < n; j++){
            cout<<s[i][j];
        }
        cout<<endl;
    }

}
int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        cin>>s[i];
    }
    search();
    draw();
    print();
    return 0;
}

```

