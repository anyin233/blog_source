---
title: AcWing 173.矩阵距离
date: 2022-01-24 13:33:19
tags: [AcWing,算法学习]

---

# 算法思路

该算法实际上是针对BFS的一种扩展写法。我们都知道BFS算法会构造一个搜索树，而如果我们希望从多个起点开始BFS最简单的方式便是使用一个虚拟的BFS起点，并在该起点后面连接所有我们希望作为起点的点即可。表现在代码中则变成了用于BFS的队列在初始化的时候将会加入所有的希望作为起点的点。

# 算法实现

```c++
#include <iostream>
#include <cstring>
#include <queue>

#define PII pair<int, int>

#define x first
#define y second

using namespace std;

const int N = 1010;

int n, m;
int g[N][N], dist[N][N], st[N][N];
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

int main()
{
    queue<PII> q;
    memset(dist, 0x3f, sizeof dist);
    
    cin >> n >> m;
    cin.get();
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < m; ++j)
        {
            char t = cin.get();
            //cout << t << ' ';
            g[i][j] = t - '0';
            if (g[i][j]) 
            {
                q.push({i, j});
                dist[i][j] = 0;
            }
        }
        cin.get();
    }
    
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        for (int i = 0; i < 4; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || y < 0 || x >= n || y >= m || st[x][y]) continue;
            dist[x][y] = min(dist[x][y], dist[t.x][t.y] + 1);
            q.push({x, y});
        }
    }
    
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < m; ++j)
        {
            cout << dist[i][j] << ' ';
        }
        cout << '\n';
    }
}
```

