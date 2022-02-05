---
title: LeetCode 1219.黄金矿工
date: 2022-02-05 11:38:18
tags: [LeetCode,每日一题]

---

# 算法思路

简单的DFS，由于题目中允许从任意点出发，故需要遍历每一个格子作为起点进行DFS。其余的和常规DFS相同

# 算法实现

``` c++
class Solution {
public:
    void dfs(int x, int y, vector<vector<int>>& grid, int st[110][110], int sum, int& res)
    {
        int dx[] = {1, -1, 0, 0};
        int dy[] = {0, 0, 1, -1};

        res = max(res, sum);
        for (int i = 0; i < 4; ++i)
        {
            int nx = dx[i] + x, ny = dy[i] + y;
            if (nx < 0 || ny < 0 || nx >= grid.size() || ny >= grid[0].size()) continue;
            if (st[nx][ny] || grid[nx][ny] == 0) continue;
            st[nx][ny] = true;
            dfs(nx, ny, grid, st, sum + grid[nx][ny], res);
            st[nx][ny] = false;
        }
    }
    int getMaximumGold(vector<vector<int>>& grid) {
        int st[110][110];
        memset(st, false, sizeof st);
        int res = 0;
        
        for (int i = 0; i < grid.size(); ++i)
        {
            for (int j = 0; j < grid[0].size(); ++j)
            {
                if (grid[i][j] == 0) continue;
                st[i][j] = true;
                dfs(i, j, grid, st, grid[i][j], res);
                st[i][j] = false;
            }
        }
        return res;
    }
};
```



 
