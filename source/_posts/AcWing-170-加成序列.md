---
title: AcWing 170.加成序列
date: 2022-02-05 22:30:14
tags: [AcWing,算法学习,DFS,迭代加深]

---

# 算法思路

迭代加深实际上是DFS中常用的一种剪枝方法。针对某些DFS的情况，其完整DFS的树非常深，但是答案却在较浅的位置时可以使用。常见的命题思路为求一个最短路。

本题采用迭代加深的算法进行，不断扩大搜索范围（深度），最终得到最短路径。

# 算法实现

```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 110;

int path[N];
bool st[N];
int n;

bool dfs(int u, int depth)
{
    if (u > depth) return false;
    if (path[u - 1] == n) return true;
    
    for (int i = u - 1; i >= 0; --i)
    {
        for (int j = i; j >= 0; --j)
        {
            int s = path[i] + path[j];
            if (s > n || s <= path[u - 1] || st[s]) continue;
            
            st[s] = true;
            path[u] = s;
            if (dfs(u + 1, depth)) return true;
            st[s] = false;
        }
    }
    
    return false;
}

int main(){
    path[0] = 1;
    while(cin >> n, n)
    {
        int depth = 1;
        memset(st, false, sizeof st);
        while(!dfs(1, depth)) depth ++;
        for (int i = 0; i < depth; ++i)
        {
            cout << path[i] << ' ';
        }
        cout << '\n';
    }
}
```

