---
title: AcWing 175.电路维修
date: 2022-01-25 11:35:20
tags: [AcWing,算法学习,双端队列,TODO]

---

#算法思路

TODO

#算法实现

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <deque>
#include <string>

#define PII pair<int, int>
#define x first
#define y second

using namespace std;

const int N = 510;

int t, n, m;
char g[N][N];
bool st[N][N];
int dist[N][N];

char way[5] = "\\/\\/";
int iy[] = {-1, 0, 0, -1}, ix[] = {-1, -1, 0, 0};
int dx[] = {-1, -1, 1, 1}, dy[] = {-1, 1, 1, -1};

int bfs()
{
    deque<PII> q;
    memset(st, false, sizeof st);
    memset(dist, 0x3f, sizeof dist);
    
    q.push_back({0, 0});
    dist[0][0] = 0;
    
    while(!q.empty())
    {
        auto t = q.front();
        q.pop_front();
        //cout << t.x << ' ' << t.y << '\n';
        if (t.x == n && t.y == m) return dist[t.x][t.y];
        
        if (st[t.x][t.y]) continue;
        st[t.x][t.y] = true;
        
        for (int i = 0; i < 4; ++i)
        {
            int x = t.x + dx[i], y = t.y + dy[i];
            if (x < 0 || y < 0 || x > n || y > m) continue;
            int gx = t.x + ix[i], gy = t.y + iy[i];
            int w = (g[gx][gy] != way[i]);
            int d = dist[t.x][t.y] + w;
            //cout << d << ' ' << x << ' ' << y << endl;
            if (d <= dist[x][y])
            {
                dist[x][y] = d;
                if (!w) q.push_front({x, y});
                else q.push_back({x, y});
            }
        }
    }
    
    return -1;
}

int main()
{
    cin >> t;
    while(t --)
    {
        cin >> n >> m;
        string temp;
        
        for (int i = 0; i < n; ++i)
        {
            cin >> temp;
            for (int j = 0; j < m; ++j) g[i][j] = temp[j];
        }
        
        if ((n + m) % 2) cout << "NO SOLUTION\n";
        else
        {
            cout << bfs() << endl;
        }
    }
}


```



 
