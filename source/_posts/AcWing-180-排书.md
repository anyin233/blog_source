---
title: AcWing 180.排书
date: 2022-02-06 21:38:01
tags: [AcWing,算法学习,DFS,IDA*]

---

# 算法思路

## IDA\*

该算法为A\*和迭代加深的结合，其中相对于A\*的估价函数额外增加了基于估价函数超过迭代层数的剪枝。

## 排书交换

本题中由于可以一次性将连续的一串书本直接插入到后面的某个位置，故可以考虑从遍历长度入手。

交换的过程可以如下所示

```
-----|-----|------|--------
     l     r      k   
```

整个交换过程可以分为下面几步

- 首先将原始的数组`b`复制一份到备用数组`w[depth]`
- 将`w[depth]`的`r + 1`到`k`的部分复制到`p`的`l`到`l + k - r `区域
- 将`w[depth]`的`l`到`r`部分复制到`p`的对应剩余部分。
- ...

在这里我们需要将`l`到`r`的书籍插入到`r + 1`的位置，可以考虑使用如下方式

```c++
int x, y;
for (int x = r + 1, y = l; x <= k; ++x, ++y) b[y] = w[depth][x];
for (int x = l; x <= r; ++x, ++y) b[y] = w[depth][x];
```

# 算法实现

```c++
#include <iostream>
#include <cstring>

using namespace std;

const int N = 16;

int b[N], n, c;
int w[5][N];

int f()
{
    int cnt = 0;
    for (int i = 0; i < n - 1; ++i)
    {
        if (b[i] != b[i + 1] - 1) cnt ++;
    }
    
    return (cnt + 2) / 3;
}

bool check()
{
    for (int i = 0; i < n - 1; ++i)
    {
        if (b[i] != b[i + 1] - 1) return false;
    }
    return true;
}

bool dfs(int u, int max_step)
{
    if (u + f() > max_step) return false;
    if (check()) return true;
    
    for (int len = 1; len <= n; ++len)
    {
        for (int l = 0; l + len - 1 < n; ++l)
        {
            int r = l + len - 1;
            for (int k = r + 1; k < n; ++k)
            {
                memcpy(w[u], b, sizeof w[u]);
                int x, y;
                for (x = r + 1, y = l; x <= k; x ++, y ++) b[y] = w[u][x];
                for (x = l; x <= r; x ++, y ++) b[y] = w[u][x];
                if (dfs(u + 1, max_step)) return true;
                memcpy(b, w[u], sizeof w[u]);
            }
        }
    }
    
    return false;
}

int main()
{
    cin >> c;
    while(c --)
    {
        cin >> n;
        for (int i = 0; i < n; ++i) cin >> b[i];
        
        int max_step = 0;
        while(max_step < 5 && !dfs(0, max_step)) max_step ++;
        
        if (max_step >= 5) cout << "5 or more\n";
        else cout << max_step << endl;
    }
}
```

