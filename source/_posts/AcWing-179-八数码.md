---
title: AcWing 179.八数码
date: 2022-01-26 14:05:51
tags: [AcWing,算法学习,A*]
---

# 算法思路

本算法实际上也是BFS的一种应用，但是朴素的BFS搜索会导致需要搜索的状态过多从而发生TLE或者MLE。为了解决这种问题故采用了A\*算法

## 估计函数

A\*算法中一个关键的部分便是估计函数。通过估计函数使得算法可以找到一个接近理想情况的最佳路径。通常情况下一个可用的估计函数必须满足估计值小于最小值，本题的估计函数选择了各个点到其最终位置的曼哈顿距离之和。

A\*算法使用一个小根堆代替了BFS中的队列，每次从小根堆中取出一个期望距离最短的点进行扩展，其中期望距离=当前点距离起点的距离+当前点到终点的**估计距离**。

# 算法实现

```c++
#include <iostream>
#include <algorithm>
#include <queue>
#include <unordered_map>
#include <string>
#include <vector>

using namespace std;

#define PIS pair<int, string>

int has_no_result(string s)
{
    int sum = 0;
    for (int i = 0; i < s.size(); ++i)
    {
        for (int j = i + 1; j < s.size(); ++j)
        {
            if (s[i] == 'x' || s[j] == 'x') continue;
            sum += (s[i] > s[j]);
        }
    }
    
    return sum & 1;
}

int f(string s)
{
    int sum = 0;
    for (int i = 0; i < s.size(); ++i)
    {
        if (s[i] == 'x') continue;
        int num = s[i] - '0';
        int x = i % 3;
        int y = i / 3;
        int acx = (num - 1) % 3;
        int acy = (num - 1) / 3;
        sum += (abs(x - acx) + abs(y - acy));
    }
    
    return sum;
}

void bfs(string s)
{
    unordered_map<string, int> dist;
    unordered_map<string, pair<char, string>> prev;
    priority_queue<PIS, vector<PIS>, greater<PIS>> heap;
    
    heap.push({f(s), s});
    dist[s] = 0;
    
    int dx[] = {0, 0, 1, -1};
    int dy[] = {1, -1, 0, 0};
    char op[] = {'d', 'u', 'r', 'l'};
    string final = "12345678x";
    
    while(!heap.empty())
    {
        auto t = heap.top();
        heap.pop();
        
        string state = t.second;
        if (state == final) break;
        int d = t.first;
        
        int x = 0, y = 0;
        for (int i = 0; i < state.size(); ++i)
        {
            if (state[i] == 'x')
            {
                x = i % 3;
                y = i / 3;
                break;
            }
        } // compute position
        
        string dummy = state;
        for (int i = 0; i < 4; ++i)
        {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx < 0 || nx >= 3 || ny < 0 || ny >= 3) continue;
            swap(dummy[y * 3 + x], dummy[ny * 3 + nx]);
            if (!dist.count(dummy) || dist[dummy] > dist[state] + 1)
            {
                dist[dummy] = dist[state] + 1;
                heap.push({f(dummy) + dist[dummy], dummy});
                prev[dummy] = {op[i], state};
            }
            
            dummy = state;
        }
    }
    
    string res;
    while(final != s)
    {
        res += prev[final].first;
        final = prev[final].second;
    }
    
    reverse(res.begin(), res.end());
    cout << res;
}

int main()
{
    string s;
    for (int i = 0; i < 9; ++i)
    {
        char t;
        cin >> t;
        s += t;
    }
    
    if (has_no_result(s))
    {
        cout << "unsolvable";
        return 0;
    }
    
    bfs(s);
}
```

