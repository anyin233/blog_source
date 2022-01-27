---
title: AcWing 1116.马走日
date: 2022-01-27 14:43:37
tags: [AcWing,算法学习,DFS]

---

# 算法思路

简单粗暴的一个DFS

# 算法实现

```c++
#include <iostream>
#include <cstring>

#define PII pair<int, int>

#define x first
#define y second

using namespace std;

const int N = 11;

int st[N][N];
int res;
int dx[] = {1, -1, 2, -2, -1, 1, -2, 2};
int dy[] = {2, -2, 1, -1, 2, -2, 1, -1};

void dfs(int n, int m, PII p, int cnt)
{
    if (cnt == n * m)
    {
        res ++;
        return;
    }
    
    st[p.x][p.y] = true;
    for (int i = 0; i < 8; ++i)
    {
        int x = p.x + dx[i], y = p.y + dy[i];
        if (x < 0 || y < 0 || x >= n || y >= m || st[x][y]) continue;
        dfs(n, m, {x, y}, cnt + 1);
    }
    
    st[p.x][p.y] = false;
}

int main()
{
    int k, n, m;
    PII start;
    
    cin >> k;
    while(k--)
    {
        res = 0;
        cin >> n >> m >> start.x >> start.y;
        dfs(n, m, start, 1);
        cout << res << '\n';
    }
}
```

