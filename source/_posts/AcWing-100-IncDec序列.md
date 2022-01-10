---
title: AcWing 100.IncDec序列
date: 2022-01-07 17:41:42
tags: [算法学习,AcWing,贪心,差分]

---

# 算法思路

算法基于查分数组的前缀和即是原数组的思想进行。

对于在原始数组上[m, n]区间每个数字+n，体现在差分数组上即第m个数字+n同时第n+1个数字-n。

---

本题提供的基本操作可以使用差分进行表示，即每次+1或-1的操作可以在差分数组上对两端的数字进行操作实现，对于整个差分数组可以用一个折线图进行表示，可以发现当所有的数字经过该操作到达相同值所需步骤最少的过程必然是所有数字达到重叠的区域内。

<!---more--->

# 算法实现

```c++
#include <iostream>

using namespace std;

const int N = 100010;

int a[N], n;
long long pos = 0, neg = 0;

int main() 
{
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    
    for (int i = n; i > 1; --i) a[i] -= a[i - 1];
    for (int i = 2; i <= n; ++i)
    {
        if (a[i] > 0) pos += a[i];
        else neg -= a[i];
    }
    
    cout << min(pos, neg) + abs(pos - neg) << '\n' << abs(pos - neg) + 1;
}
```



