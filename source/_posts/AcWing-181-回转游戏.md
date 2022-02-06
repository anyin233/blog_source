---
title: AcWing 181.回转游戏
date: 2022-02-06 22:48:11
tags: [AcWing,算法学习,DFS,IDA*]

---

# 算法思路

该算法基本大体框架同AcWing.180，但是题目中涉及到的8种操作需要一定的处理技巧

## 八种回转操作

首先可以考虑对八种操作进行硬编码，此处采用顺时针从0到7。然后由于题目中传入的数字序列并非便于操作的方式，我们可以进行一个打表操作，提前确定会被影响到的位置。

```
         A     B
         0     1
         2     3
 H 4  5  6  7  8  9  10 C
         11    12      
 G 13 14 15 16 17 18 19 D
         20    21
         22    23
         F     E
```

即可以生成一个`op`数组

```c++
int op[8][7] = {
  {0, 2, 6, 11, 15, 20, 22}, // A
  {1, 3, 8, 12, 17 ,21, 23}, // B
  {10, 9, 8, 7, 6, 5, 4}, // C
  {19, 18, 17, 16, 15, 14, 13}, // D
  {23, 21, 17, 12, 8, 3, 1}, // E
  {22, 20, 15, 11, 6, 2, 0}, // F
  {13, 14, 15, 16, 17, 18, 19}, // G
  {4, 5, 6, 7, 8, 9, 10}, // H
};
```

同样可以生成一个相反操作的数组，防止出现两次相反的操作导致无效操作

```c++
int opposite = {5, 4, 7, 6, 1, 0, 3, 2};
```

# 算法实现

```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 24;

int op[8][7] = {
  {0, 2, 6, 11, 15, 20, 22}, // A
  {1, 3, 8, 12, 17 ,21, 23}, // B
  {10, 9, 8, 7, 6, 5, 4}, // C
  {19, 18, 17, 16, 15, 14, 13}, // D
  {23, 21, 17, 12, 8, 3, 1}, // E
  {22, 20, 15, 11, 6, 2, 0}, // F
  {13, 14, 15, 16, 17, 18, 19}, // G
  {4, 5, 6, 7, 8, 9, 10}, // H
};

int oppsite[] = {5, 4, 7, 6, 1, 0, 3, 2};
int center[] = {6, 7, 8, 11, 12, 15, 16, 17};

int n;
int g[N];
int path[100];

int f()
{
    static int sum[4];
    memset(sum, 0, sizeof sum);
    
    for (int i = 0; i < 8; ++i) sum[g[center[i]]] ++;
    
    int s = 0;
    for (int i = 1; i <= 3; ++i) s = max(s, sum[i]);
    
    return 8 - s;
}

void move(int ac)
{
    int t = g[op[ac][0]];
    for (int i = 0; i < 6; ++i) g[op[ac][i]] = g[op[ac][i + 1]];
    g[op[ac][6]] = t;
}

bool dfs(int u, int depth, int last)
{
    if (u + f() > depth) return false;
    if (f() == 0) return true;
    
    for (int i = 0; i < 8; ++i)
    {
        if (oppsite[i] != last) // prevent it goes back
        {
            move(i);
            path[u] = i;
            if (dfs(u + 1, depth, i)) return true;
            move(oppsite[i]);
        }
    }
    
    return false;
}

int main()
{
    while(cin >> g[0], g[0])
    {
        for (int i = 1; i < N; ++i) cin >> g[i];
        
        int depth = 0;
        while(!dfs(0, depth, -1)) depth ++;
        
        if (!depth) cout << "No moves needed";
        else 
        {
            for (int i = 0; i < depth; ++i) cout << (char) (path[i] + 'A');
        }
        cout << '\n' << g[center[0]] << '\n';
    }
}
```

