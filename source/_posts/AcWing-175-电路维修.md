---
title: AcWing 175.电路维修
date: 2022-01-25 11:35:20
tags: [AcWing,算法学习,双端队列,TODO]

---

# 算法思路

本算法实际上也是宽搜的一种应用形式。

题目中我们希望通过旋转某几个电路使得整个电路联通，由于所有的电路均为对角线，故我们可以知道如果希望电路联通，终点座标和和起点座标和的奇偶性相同。故由此我们可以直接判断得出无解的情况。

而当有解的情况下，我们可以将**电路被旋转**视作边权1、**电路未被旋转**视作边权0，题目将会被转化为一个最短路问题。基于该最短路问题我们可以写出下面代码中的BFS逻辑，即每次扩展从一个点出发向其可能可以到达的四个点进行扩展，对应会有四种不同的边，扩展所使用的边如果和题目中所给出的边不同则为**被旋转**，设置为边权1，否则为0。

由此和常规的BFS不同的一点在于，该BFS所扩展的边权可能不同，所以我们可以考虑采用**双端队列**，即扩展边权为0的放在队列前端，扩展边权为1的放在边权后端。由于每次从队列中选择的均为队列中距离最短的边，且队列中必然只存在$x$和$x+1$两种不同距离的点，所以说使用双端队列同样满足BFS的要求。

# 算法实现

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



 
