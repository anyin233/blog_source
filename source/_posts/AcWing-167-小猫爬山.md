---
title: AcWing 167.小猫爬山
date: 2022-01-29 17:18:59
tags: [AcWing,算法学习,DFS,剪枝]

---

# 算法思路

考虑到本算法的时间复杂度，如果遍历全部的情况会导致TLE，故需要进行一定的剪枝。

所以本题需要进行适当的剪枝，优先搜索状态较少的方案，即从个体质量较大的单位开始搜索。同时对非最优方案进行剪枝，最终可以得到结果

# 算法实现

```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 20;
int n, w;
int res;
int c[N], group[N];

void dfs(int t, int g)
{
    if (g > res) return; // 非最优剪枝
    if (t == n) 
    {
        res = g;    
        return;
    }
    
    
    for (int i = 0; i < g; ++i)
    {
        if (c[t] + group[i] <= w) //非可行方案剪枝
        {
            group[i] += c[t];
            dfs(t + 1, g);
            group[i] -= c[t];
        }
    }
    
    group[g] += c[t];
    dfs(t + 1, g + 1);
    group[g] = 0;
}

int main()
{
    res = 0x3f3f3f3f;
    cin >> n >> w;
    for (int i = 0; i < n; ++i) cin >> c[i];
    sort(c, c + n);
    reverse(c, c + n);
    
    dfs(0, 1);
    
    cout << res;
}
```

