---
title: AcWing 178.第K短路
date: 2022-01-26 15:08:59
tags: [AcWing,算法学习,A*]

---

# 算法思路

本题利用了A\*的一个性质，即当终点第N次弹出的时候得到的就是第N小的值，其余与常规A\*解法一致。

# 代码实现

```c++
#include <iostream>
#include <queue>
#include <algorithm>
#include <vector>
#include <cstring>

#define PII pair<int, int>
#define PIII pair<int, pair<int, int>>

using namespace std;

const int N = 1010, M = 20010;

int h[N], rh[N], e[M], ne[M], w[M], dist[N];
bool st[N];
int n, m, S, K, T, idx;

void add(int h[N], int a, int b, int c)
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

void dijkstra()
{
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    heap.push({0, T});
    memset(dist, 0x3f, sizeof dist);
    
    dist[T] = 0;
    while(!heap.empty())
    {
        auto t = heap.top();
        heap.pop();
        
        int num = t.second, distance = t.first;
        if (st[num]) continue;
        st[num] = true;
        
        for (int i = rh[num]; ~i; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > distance + w[i])
            {
                dist[j] = distance + w[i];
                heap.push({dist[j], j});
            }
        }
    }
}

int astar()
{
    priority_queue<PIII, vector<PIII>, greater<PIII>> heap;
    heap.push({dist[S], {0, S}});
    if (dist[0] == dist[S]) return -1;
    int cnt = 0;
    while(!heap.empty())
    {
        auto t = heap.top();
        heap.pop();
        
        int n = t.second.second, distance = t.second.first;
        if (n == T) cnt++;
        if (cnt == K) return distance;
        for (int i = h[n]; ~i; i = ne[i])
        {
            int j = e[i];
            heap.push({distance + w[i] + dist[j], {distance + w[i], j}});
        }
    }
    
    return -1;
}

int main()
{
    memset(h, -1, sizeof h);
    memset(rh, -1, sizeof rh);
    
    cin >> n >> m;
    for (int i = 0; i < m; ++i)
    {
        int a, b, c;
        cin >> a >> b >> c;
        add(h, a, b, c);
        add(rh, b, a, c);
    }
    
    cin >> S >> T >> K;
    if (S == T) K++;
    dijkstra();
    
    cout << astar();
}
```

