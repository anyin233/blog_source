---
title: AcWing 1098.城堡问题
date: 2022-01-19 15:58:19
tags: [AcWing,算法学习,搜索]
---

# 算法思路

本题实际上不难，和1097解法相同，唯一需要注意的问题是本题使用多个数字之和表示每个房间的墙的情况。但是根据题意实际上就是用一个四位二进制表示，直接按位判断就可以。

# 算法实现

```c++
#include <iostream>
#include <queue>

using namespace std;

const int N = 55;

int g[N][N], v[N][N];
int n, m;
int x[] = {0, -1, 0, 1};
int y[] = {-1, 0, 1, 0};


int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= m; ++j)
        {
            cin >> g[i][j];
        }
    }
    
    queue<pair<int, int>> visited;
    int cnt = 0, mxar = 0;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= m; ++j)
        {
            int arr = 0;
            if (!v[i][j])
            {
                cnt ++;
                visited.push({i, j});
            }
            
            while(!visited.empty())
            {
                int ox = visited.front().first, oy = visited.front().second;
                visited.pop();
                if (v[ox][oy]) continue;
                v[ox][oy] = 1;
                arr ++;
                
                for (int s = 0; s < 4; ++s)
                {
                    int nx = ox + x[s], ny = oy + y[s];
                    if (nx > n || ny > m || nx < 1 || ny < 1) continue;
                    if (!(g[ox][oy] >> s & 1))
                    {
                        visited.push({nx, ny});
                        //cout << nx << ' ' << ny << '\n';
                    }
                }
            }
            
            mxar = max(mxar, arr);
        }
    }
    
    cout << cnt << '\n' << mxar;
}
```

