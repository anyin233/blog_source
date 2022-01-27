---
title: AcWing 1118.分成互质组
date: 2022-01-27 16:35:08
tags: [AcWing,算法学习,DFS]
---

# 算法思路

本算法的基本思路不难，但是必须要注意实现分组的方式不能写错，只需要简单地记录每个组所保存的数字的索引即可得到答案。

# 算法实现

```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 101;

int n, res;
int a[N], group[N][N];
bool st[N];

int gcd(int a, int b)
{
    if (b == 0) return a;
    return gcd(b, a % b);
}

bool check(int group[], int gc, int i)
{
    for (int j = 0; j < gc; ++j)
    {
        if (gcd(a[group[j]], a[i]) > 1) return false;
    }
    return true;
}

void dfs(int g, int gc, int tc, int i)
{
    if (g >= res) return;
    if (tc == n) res = g;
    //cout << tc << endl;
    
    bool flag = true;
    for (int j = i; j < n; ++j)
    {
        if (!st[j] && check(group[g], gc, j))
        {
            st[j] = true;
            group[g][gc] = j;
            
            dfs(g, gc + 1, tc + 1, j + 1);
            st[j] = false;
            flag = false;
        }
    }
    
    if (flag)
    {
        dfs(g + 1, 0, tc, 0);
    }
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; ++i) cin >> a[i];
    res = 0x3f3f3f3f;
    //group[0][0] = a[0];
    dfs(1, 0, 0, 0);
    
    cout << res;
}
```

