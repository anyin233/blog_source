---
title: AcWing 91.最短Hamilton路径
date: 2022-01-07 16:21:12
tags: [算法学习,AcWing]

---

# 算法逻辑

本题采用状态压缩DP方式实现。

定义状态`i`为一个二进制串，其中1代表已访问的点，0代表未访问的点。DP数组定义为  当前状态*当前正在访问的点。

# 代码实现

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <climits>

using namespace std;

const int N = 22, M = 1 << 20;

int g[N][N], dp[M][N];
int m, n;

int main() 
{
    fill(dp[0], dp[0] + M * N, 0x3f3f3f3f);
    dp[1][0] = 0; // initial_data
    
    cin >> n;
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < n; ++j)
        {
            cin >> g[i][j];
        }
    }
    // read graph
    
    for (int i = 0; i < (1 << n); ++i) // for each status i
    {
        for (int j = 0; j < n; ++j) 
        {
            if (i >> j & 1) // arrived j
            {
                for (int k = 0; k < n; ++k)
                {
                    if ((i - (1 << j)) >> k & 1) // arrived k but not arrived j
                    {
                        dp[i][j] = min(dp[i][j], dp[i - (1 << j)][k] + g[j][k]);
                    }
                }
            }
        }
    }
    
    cout << dp[(1 << n) - 1][n - 1];
}
```



 
