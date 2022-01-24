---
title: AcWing 1107.魔板
date: 2022-01-24 15:31:53
tags: [AcWing,算法学习]

---

# 算法思路

本题实际上依然是BFS的一种扩展方式。题目中要求从初始状态转换到一个特定的目标状态，同时转换操作固定，故可以考虑采用BFS进行搜索，搜索得到的必然是最少操作的方式。

同时本题要求记录操作序列，故可以参考*1076.迷宫问题*中记录前驱操作的思想记录每一个操作，最终通过迭代得到操作序列。

# 算法实现

```c++
#include <iostream>
#include <unordered_map>
#include <string>
#include <queue>
#include <algorithm>

using namespace std;

unordered_map<string, int> dist;
unordered_map<string, pair<char, string>> pre; // <Current, <Operation, LastState>>
queue<string> q;
string target, start="12345678";


string move0(string a)
{
    reverse(a.begin(), a.end());
    return a;
}

string move1(string a)
{
    string temp;
    temp += a[3];
    for (int i = 0; i < 3; ++i)
    {
        temp += a[i];
    }
    
    for (int i = 5; i < 8; ++i)
    {
        temp += a[i];
    }
    
    temp += a[4];
    return temp;
}

string move2(string a)
{
    char t1 = a[1], t2 = a[2], t3 = a[5], t4 = a[6];
    a[1] = t4;
    a[2] = t1;
    a[5] = t2;
    a[6] = t3;
    return a;
}

void bfs(string start, string target)
{
    q.push(start);
    dist[start] = 0;
    
    while(!q.empty())
    {
        auto t = q.front();
        q.pop();
        
        string m[3];
        m[0] = move0(t);
        m[1] = move1(t);
        m[2] = move2(t); // generate string by Op.A B C
        
        for (int i = 0; i < 3; ++i)
        {
            if (dist[m[i]] != 0 || m[i] == start) continue;
            
            dist[m[i]] = dist[t] + 1;
            pre[m[i]] = {'A' + i, t};
            
            if (m[i] == target) break;
            
            q.push(m[i]);
        }
    }
}


int main()
{
    for (int i = 0; i < 8; ++i)
    {
        int t;
        cin >> t;
        target += char(t + '0');
    }
    
    bfs(start, target);
    
    cout << dist[target] << '\n';
    
    string res;
    while(target != start)
    {
        res += pre[target].first;
        target = pre[target].second;
    }
    reverse(res.begin(), res.end());
    
    if (res.size()) cout << res;
}
```

