---
title: AcWing 1106.山峰和山谷
date: 2022-01-19 17:40:44
tags: [AcWing,算法学习,搜索]

---

# 算法思路

本题实际上只是在1097和1098的基础上增加了少量的额外操作。对于判断山峰山谷的操作只需要在确定连通域的同时确定在该连通域边界是否存在更高或更低的格子再进行判断。

# 代码实现

```c++
#include <iostream>
#include <queue>



using namespace std;

const int N = 1010;

int g[N][N], st[N][N];
int n;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    cin >> n;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= n; ++j)
        {
            cin >> g[i][j];
        }
    }
    
    queue<pair<int, int>> q;
    int peak = 0, vally = 0;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= n; ++j)
        {
            bool has_higher = false, has_lower = false;
            if (!st[i][j])
            {
                q.push({i, j});
            }
            
            while(!q.empty())
            {
                auto t = q.front();
                q.pop();
                if (st[t.first][t.second]) continue;
                st[t.first][t.second] = 1;
                for (int x = t.first - 1; x <= t.first + 1; ++x)
                {
                    for (int y = t.second - 1; y <= t.second + 1; ++y)
                    {
                        if (x < 1 || y < 1 || x > n || y > n) continue;
                        if (g[x][y] > g[t.first][t.second]) has_higher = true;
                        else if (g[x][y] < g[t.first][t.second]) has_lower = true;
                        else q.push({x, y});
                    }
                }
            }
            
            if (!(has_lower && has_higher) && (has_higher || has_lower))
            {
                if (has_higher) vally ++;
                if (has_lower) peak ++;
            }
        }
    }
    
    if (peak || vally)
        cout << peak << ' ' << vally;
    else cout << "1 1";
}
```

