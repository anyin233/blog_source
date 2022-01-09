---
title: AcWing 1068. 环形石子合并
date: 2022-01-09 16:41:02
tags: [算法学习,AcWing,区间DP,DP,区间和]

---

# 算法思路

对于石子合并，对于第$i$和第$i + 1$两堆石子进行合并，将会得到得分$a[i] + a[i + 1]$。

即可以考虑对于第$1...n$号石子，最后一次合并的得分必然为所有石子数量的总和，即可以选择一个中间位置$i$，即第$i$次合并后的总得分为$sc[1...i] + sc[i + 1...n] + a[1 ... n]$即为前半段的得分加上后半段的得分再加上当前石子数量的总和。

针对求和问题，可以考虑直接求区间和，即有

```c++
a[i] += a[i - 1]
```

则对于第$i$到第$j$的总和为$a[j] - a[i - 1]$。

而对于划分问题，可以考虑dp数组的定义，`dp[i][j]`为第$i$到第$j$号石子合并的得分。得到转移方程

```c++
dp[i][j] = max(dp[i][j], d[i][k] + dp[k + 1][j]);
```

同时对于区间DP问题，我们需要明确需要从小区间向大区间遍历，即`for`循环遍历的过程是分别遍历**区间长度**和**区间左端点**，并使用二者计算得到区间右端点。

另外本题中的数组为一个循环数组，为了实现循环数组中的区间DP，可以考虑将数组重复一遍并限定区间DP的长度控制在`n`，实现对于环形数组的处理

#代码实现

```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 420;

int n, a[N], dpmx[N][N], dpmn[N][N];

int main() 
{
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i], a[i + n] = a[i];
    
    for (int i = 2; i <= n << 1; ++i) a[i] += a[i - 1];//计算区间和
    
    memset(dpmn, 0x3f, sizeof dpmn);//初始化最小值数组
    
    for (int len = 1; len <= n; ++len)
    {
        for (int l = 1, r; r = l + len - 1, r < n << 1; ++l)
        {
            if (len == 1) dpmx[l][l] = dpmn[l][l] = 0;
            else
            {
                for (int k = l; k < r; ++k)
                {
                    dpmx[l][r] = max(dpmx[l][r], dpmx[l][k] + dpmx[k + 1][r] + a[r] - a[l - 1]);
                    dpmn[l][r] = min(dpmn[l][r], dpmn[l][k] + dpmn[k + 1][r] + a[r] - a[l - 1]);
                }
            }
        }
    }
    
    int maxv = -0x3f3f3f3f;
    int minv = 0x3f3f3f3f;
    
    
    for (int l = 1; l <= n; ++l)
    {
        maxv = max(maxv, dpmx[l][l + n - 1]);
        minv = min(minv, dpmn[l][l + n - 1]);
    }
    
    cout << minv << '\n' << maxv;
}
```



