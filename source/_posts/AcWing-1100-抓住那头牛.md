---
title: AcWing 1100.抓住那头牛
date: 2022-01-24 11:07:42
tags: [AcWing,算法学习,搜索]

---

# 算法思路

这道题更为简单粗暴，相比188和1076的二维搜索问题，本题将搜索放到了一维层面，实际上更加简单了，具体思路同dijkstra模板

# 算法实现

```c++
#include <iostream>
#include <queue>
#include <cstring>

using namespace std;

const int N = 100010;

int n, k;
int dist[N], st[N];

int main()
{
    cin >> n >> k;
    
    queue<int> q;
    q.push(n);
    
    memset(dist, 0x3f, sizeof dist);
    dist[n] = 0;
    
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        if (st[t]) continue;
        st[t] = true;
        if (t - 1 >= 0) 
        {
            dist[t - 1] = min(dist[t - 1], dist[t] + 1);
            q.push(t - 1);
        }
        
        if (t + 1 < N)
        {
            dist[t + 1] = min(dist[t + 1], dist[t] + 1);
            q.push(t + 1);
        }
        
        if (t * 2 < N)
        {
            dist[t * 2] = min(dist[t * 2], dist[t] + 1);
            q.push(t * 2);
        }
    }
    
    cout << dist[k];
}
```

