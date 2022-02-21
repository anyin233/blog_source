---
title: AcWing 343.排序
date: 2022-02-21 21:30:35
tags: [AcWing,算法学习,floyd,传递闭包]

---

# 算法思路

本题的基本思路为基于floyd算法实现传递闭包，即将所有的大小关系进行传递，得到一个全局的大小关系，基于该大小关系判断是否存在矛盾的情况。

矛盾的情况有两种

- `A < A && A > A`
- `A < B && B < A`

二者可以在构造整个大小关系图的过程中被逐渐排查出来，即可以一边建图一边检查，当其中出现了任何一个冲突的时候便会终止后续的检查操作，大幅度减少计算量。

对于最终输出序列的方法，考虑到本题的数据规模较小，可以考虑直接使用暴力枚举的方式，枚举每一个大小关系并标记，直接得到序列。

# 算法实现

```c++
#include <iostream>
#include <cstring>

using namespace std;

const int N = 30;

int g[N][N];
int n, m;
bool st[N];

void floyd()
{
    for (int k = 0; k < n; ++k)
    {
        for (int i = 0; i < n; ++i)
        {
            for (int j = 0; j < n; ++j)
            {
                g[i][j] |= g[i][k] && g[k][j];
            }
        }
    }
}

int check()
{
    for (int i = 0; i < n; ++i) 
        if (g[i][i]) return 2;
        
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < i; ++j)
            if (!g[i][j] && !g[j][i])
            {
                return 0;    
            }
            
    return 1;
}

int get_min()
{
    for (int i = 0; i < n; ++i)
    {
        if (!st[i])
        {
            bool flag = true;
            for (int j = 0; j < n; ++j)
            {
                if (!st[j] && g[j][i])
                {
                    flag = false;
                    break;
                }
            }
            if (flag)
            {
                st[i] = true;
                return i;
            }
        }
    }
    
    return 0;
}

int main()
{
    while(cin >> n >> m, n || m)
    {
        memset(g, 0, sizeof g);
        memset(st, false, sizeof st);
        int typ = 0, t;
        for (int i = 0;  i < m; ++i)
        {
            char a[4];
            cin >> a;
            
            
            if (!typ)
            {
                g[a[0] - 'A'][a[2] - 'A'] = 1;
                floyd();
                typ = check();
                if (typ) t = i + 1;
            }
        }
        if (!typ) cout << "Sorted sequence cannot be determined.\n";
        else if (typ == 2) cout << "Inconsistency found after "<< t << " relations.\n";
        else
        {
            cout << "Sorted sequence determined after "<< t<< " relations: ";
            for (int i = 0; i < n; ++i) cout << char('A' + get_min());
            cout << ".\n";
        }
    }
    
    
}
```

