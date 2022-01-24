---
title: AcWing 1076.迷宫问题
date: 2022-01-24 10:02:29
tags: [AcWing,算法学习,BFS]
---

# 算法思路

本题实际上就是Dijkstra算法的一个小扩展，需要加入对于路径记录的支持。故可以考虑开辟一个`pre[][]`数组保存当前点需要走的下一步，通过迭代得到从起点到终点的路径即可。



# 算法实现

```c++
#include <iostream>
#include <queue>

using namespace std;

const int N = 1010, M = N * N;

#define x first
#define y second

#define PII pair<int, int>

int g[N][N];
bool st[N][N];
PII pre[N][N];
int n;

int dx[] = {0, 1, 0, -1};
int dy[] = {1, 0, -1, 0};

int main()
{
    cin >> n;
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < n; ++j)
        {
            cin >> g[i][j];
        }
    }
    
    queue<PII> q, tq;
    PII tep = {-1, -1};
    fill(pre[0], pre[0] + n * n, tep);
    
    q.push({n - 1, n - 1});
    //st[n - 1][n - 1] = true;
    bool flag = true;

    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        //cout << t.x << ' ' << t.y << '\n';
        for (int i = 0; i < 4; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || y < 0 || x >= n || y >= n || g[x][y] || st[x][y]) continue;
            pre[x][y] = t;
            q.push({x, y});
        }
        
    }

    
    PII end = {0, 0};
    cout << 0 << ' ' << 0 << endl;

    while(end.x != n - 1 || end.y != n - 1)
    {
        printf("%d %d\n", pre[end.x][end.y].x, pre[end.x][end.y].y);
        int x = end.x, y = end.y;
        end.x = pre[x][y].x, end.y = pre[x][y].y;
    }
    // for (int i = 0; i < n; ++i)
    // {
    //     for (int j = 0; j < n; ++j)
    //     {
    //         cout << '[' << pre[i][j].x << ',' << pre[i][j].y << "] ";
    //     }
    //     cout << '\n';
    // }
    
}
```

