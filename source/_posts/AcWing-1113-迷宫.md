---
title: AcWing 1113.迷宫
date: 2022-01-26 22:58:12
tags: [AcWing,算法学习,BFS]

---

# 算法思路

非常常规的一个BFS问题，只需要在搜索到终点的时候进行跳出即可，若最终未搜索到终点则返回无法到达。

# 算法实现

```c++
#include <iostream>
#include <queue>
#include <cstring>

#define PII pair<int, int>

#define x first
#define y second

using namespace std;

const int N = 110;

int k, n;
PII start, final;
char g[N][N];
bool st[N][N];

bool bfs()
{
    int dx[] = {0, 0, 1, -1};
    int dy[] = {1, -1, 0, 0};
    
    queue<PII> q;
    q.push(start);
    
    memset(st, false, sizeof st);
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        
        if (g[t.x][t.y] == '#') continue;
        if (t == final) return true;
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        for (int i = 0; i < 4; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || y < 0 || x >= n || y >= n) continue;
            q.push({x, y});
        }
    }
    
    return false;
}

int main()
{
    cin >> k;
    while(k --)
    {
        cin >> n;
        for (int i = 0; i < n; ++i)
        {
            for (int j = 0; j < n; ++j)
            {
                cin >> g[i][j];
            }
        }
        
        cin >> start.x >> start.y >> final.x >> final.y;
        if (bfs()) cout << "YES\n";
        else cout << "NO\n";
    }
}
```

