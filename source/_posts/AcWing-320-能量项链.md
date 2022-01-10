---
title: AcWing 320.能量项链
date: 2022-01-10 14:42:39
tags: [算法学习,AcWing,DP,区间DP]

---

# 算法思路

本题是一道经典的区间DP问题。

对于一串珠子$(i_1, j_1), (i_2, j_2)...(i_n, j_n)$假设对其中任意一个区间进行合并，例如对$(i_k, j_k)...(i_s, j_s)$这一段的所有珠子进行合并，最后一次合并的值（释放的能量）取决于合并的顺序。但是同样可以考虑对合并点进行枚举得到结果。

考虑当合并区间首尾确定的情况下，该次合并的结果只与其的合并的中心点的值有关，故可以考虑在区间DP的过程中枚举合并点得到结果。

而考虑到本题每个被合并的珠子具有两个参数，合并的过程可以被视作$i_k, j_k(i_{k+1}), j_{k + 1} \to i_k, j_{k + 1}$而中间的元素将会被两边共用，问题可以从合并n个带有两个参数的珠子转化为合并n+1个带有一个参数的石子的过程，且被选为分割点的石子将会被两边共用。

<!---more--->

# 算法实现

```c++
#include <iostream> 
#include <algorithm>

using namespace std;

const int N = 220;

int a[N], dp[N][N];
int n;

int main() 
{
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i], a[i + n] = a[i];
    
    for (int len = 1; len <= n + 1; ++len)
    {
        for (int l = 1, r; r = l + len - 1, r <= n << 1; ++l)
        {
            if (len == 1) dp[l][l] = 0;
            else
            {
                for (int k = l + 1; k < r; ++k)
                {
                    dp[l][r] = max(dp[l][r], dp[l][k] + dp[k][r] + a[l] * a[r] * a[k]);
                }
            }
        }
    }
    
    int maxv = 0;
    for (int l = 1; l <= n; ++l)
    {
        maxv = max(maxv, dp[l][l + n]);
    }
    
    cout << maxv;
}
```

