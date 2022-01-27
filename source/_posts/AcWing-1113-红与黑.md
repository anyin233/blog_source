---
title: AcWing 1113.红与黑
date: 2022-01-27 13:04:41
tags: [AcWing,算法学习,BFS,Flood Fill]

---

# 算法思路

常规的洪泛算法解决的问题，只需要从当前点开始扩展，对所有可以访问到的点进行计数即可

# 算法实现

```c++
#include <iostream>
#include <cstring>
#include <queue>

#define PII pair<int, int>

#define x first
#define y second

using namespace std;

const int N = 21;

int n, m;
char g[N][N];
bool st[N][N];
PII start;

int bfs()
{
    queue<PII> q;
    int cnt = 0;
    
    q.push(start);
    
    int dx[] = {1, -1, 0, 0};
    int dy[] = {0, 0, 1, -1};
    
    memset(st, false, sizeof st);
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        cnt ++;
        
        for (int i = 0; i < 4; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || x >= n || y < 0 || y >= m || g[x][y] == '#') continue;
            q.push({x, y});
        }
    }
    
    return cnt;
}

int main()
{
    cin >> m >> n;
    while(n != 0 && m != 0)
    {
        for (int i = 0; i < n; ++i)
        {
            for (int j = 0; j < m; ++j)
            {
                cin >> g[i][j];
                if (g[i][j] == '@')
                {
                    start.x = i;
                    start.y = j;
                    g[i][j] = '.';
                }
            }
        }
        
        cout << bfs() << '\n';
        cin >> m >> n;
    } 
}
```

