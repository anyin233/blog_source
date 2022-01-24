---
title: AcWing 188.武士风度的牛
date: 2022-01-24 10:51:27
tags: [AcWing,算法学习,搜索]

---

# 算法思路

还是熟悉的Dijkstra，但是这次的行为模式变成了象棋中🐎的行进方式罢了。

# 算法实现

```c++
#include <iostream>
#include <cstring>
#include <queue>

#define PII pair<int, int>

#define x first
#define y second

using namespace std;

const int N = 160;

int g[N][N], n, m, dist[N][N], st[N][N];
PII init, target;

void bfs()
{
    queue<PII> q;
    q.push(init);
    
    int dx[] = {1, -1, 1, -1, 2, -2, 2, -2};
    int dy[] = {2, -2, -2, 2, 1, -1, -1, 1};
    
    memset(dist, 0x3f, sizeof dist);
    memset(st, 0, sizeof st);
    
    dist[init.x][init.y] = 0;
    
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        for (int i = 0; i < 8; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || y < 0 || x >= n || y >= m || g[x][y]) continue;
            dist[x][y] = min(dist[x][y], dist[t.x][t.y] + 1);
            q.push({x, y});
        }
    }
}

int main()
{
    cin >> m >> n;
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < m; ++j)
        {
            char t;
            cin >> t;
            if (t == '*') g[i][j] = 1;
            else 
            {
                g[i][j] = 0;
                if (t == 'K') init = {i, j};
                else if (t == 'H') target = {i, j};
            }
        }
    }
    
    bfs();
    
    cout << dist[target.x][target.y];
}
```

