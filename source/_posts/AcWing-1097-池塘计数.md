---
title: AcWing 1097.池塘计数
date: 2022-01-19 14:42:14
tags: [AcWing,算法学习,搜索]
---

# 题目思路

本题目较为简单，但是题目给出的数据若采用递归进行实现会导致`stack overflow`，故选择使用迭代算法实现。即采用BFS的思想进行计算，通过一个队列存储当前待访问的点并持续进行入队出队操作最终完成洪泛。

# 实现代码

```c++
#include <iostream>
#include <queue>

using namespace std;

const int N = 1010;

int g[N][N];
int n, m;
int x[] = {1, 0, -1, 0, 1, -1, 1, -1};
int y[] = {0, 1, 0, -1, 1, -1, -1, 1};

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= m; ++j)
        {
            char t;
            cin >> t;
            if (t == '.') g[i][j] = 0;
            else g[i][j] = 1;
        }
    }
    
    queue<pair<int, int>> visited;
    int cnt = 0;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= m; ++j)
        {
            if (g[i][j])
            {
                visited.push({i, j});
                g[i][j] = 0;
                cnt ++;
            }
            
            while(!visited.empty())
            {
                int ox = visited.front().first, oy = visited.front().second;
                visited.pop();
                for (int s = 0; s < 8; ++s)
                {
                    int nx = ox + x[s];
                    int ny = oy + y[s];
                    if (g[nx][ny]) visited.push({nx, ny});
                    g[nx][ny] = 0;
                }
            }
        }
    }
    
    cout << cnt;
}
```

