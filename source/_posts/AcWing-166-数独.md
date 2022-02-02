---
title: AcWing 166.数独
date: 2022-02-02 11:35:41
tags: [AcWing,算法学习,DFS,剪枝]

---

# 算法思路

## 数据表示

本题中采用单行表示一个问题，考虑采用数组记录每行、每列和每九宫格的数字出现情况。

题目中选择使用x和y计算得到每个单元格在给出的数组中对应下标。

## 剪枝思路

首先本题中每一种情况均唯一，并无冗余。采用的剪枝方法包括

- 可行性剪枝
- 优先遍历最少方案

其中可行性剪枝采用了位运算进行优化，通过位运算和lowbit算法可以在o(1)的时间内确定数独中某一行、列和九宫格存在的一个方案。同时通过提前计算每个状态所包含的空格的数量降低了在dfs过程中由于计算空格数量所带来的额外时间开销。

# 代码实现

```c++
#include <iostream>
#include <algorithm>
#include <cstring>
#include <string>
#include <bitset>

using namespace std;

const int N = 9, M = 1 << N;

int row[N], col[N], grid[3][3];
int ones[M], lg[M]; // ones保存每个状态的空格数量，其中1表示空格，lg则是一个2^n的log值预处理
string puzzle;

int lowbit(int x)
{
    return x & -x;
}

void draw(int x, int y, int t, bool is_set) // is_set为true的时候为填入数字，否则为还原为空格
{
    if (is_set) puzzle[x * N + y] = '1' + t;
    else puzzle[x * N + y] = '.';
    
    int v = 1 << t; // 此处还是应当注意本题中选择1表示空格，0表示填数
    if (!is_set) v = -v;
    row[x] -= v;
    col[y] -= v;
    grid[x / 3][y / 3] -= v;
}

void init() // 初始化使得全图均为空格
{
    for (int i = 0; i < N; ++i) row[i] = col[i] = (1 << N) - 1;
    for (int i = 0; i < 3; ++i)
    {
        for (int j = 0; j < 3; ++j)
        {
            grid[i][j] = (1 << N) - 1;
        }
    }
}

int get(int x, int y)
{
    return row[x] & col[y] & grid[x / 3][y / 3]; // 对行、列和九宫格的可行情况进行一个按为与，得到所有可行的方案
}

bool dfs(int cnt)
{
    if (cnt == 0) return true;
    
    int minv = 10, x = 0, y = 0;
    for (int i = 0; i < N; ++i)
        for (int j = 0; j < N; ++j)
            if (puzzle[i * N + j] == '.')
            {
                int state = get(i, j);
                
                if (ones[state] < maxv)
                {
                    x = i, y = j;
                    minv = ones[state];
                } //找到可选方案最少的一个位置，剪枝的其中一部分
            }
            
    int state = get(x, y);
    for (int i = state; i != 0; i -= lowbit(i))
    {
        int t = lg[lowbit(i)];
        draw(x, y, t, true);
        if (dfs(cnt - 1)) return true;
        draw(x, y, t, false);
    }
    
    return false;
}

int main()
{
    for (int i = 0; i < N; ++i) lg[1 << i] = i;
    for (int i = 0; i < 1 << N; ++i)
    {
        for (int j = 0; j < N; ++j)
        {
            ones[i] += (i >> j) & 1;
        }
    } // 状态预处理，ones记录每个状态包含1的数量
    while(cin >> puzzle)
    {
        if (puzzle == "end") break;
        
        init();
        int cnt = 0;
        for (int i = 0; i < N; ++i)
        {
            for (int j = 0; j < N; ++j)
            {
                if (puzzle[i * N + j] != '.')
                {
                    int v = puzzle[i * N + j] - '1';
                    draw(i, j, v, true);
                }
                else
                {
                    cnt ++;
                }
            }
        }
        
        dfs(cnt);
        cout << puzzle << '\n';
    }
}
```



