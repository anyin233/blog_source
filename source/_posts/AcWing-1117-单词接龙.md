---
title: AcWing 1117.单词接龙
date: 2022-01-27 15:10:30
tags: [AcWing,算法学习,DFS]
---

# 算法思路

常规的DFS问题，只需要注意如何实现字符串匹配的问题即可

# 算法实现

```c++
#include <iostream>
#include <string>

using namespace std;

const int N = 21;

string s[N];
int cnt[N];
int res;
char start;
int n;

void dfs(string last, int l)
{
    //cout << last << endl;
    res = max(l, res);
    for (int i = 0; i < n; ++i)
    {
        if (cnt[i] >= 2) continue;   
        for (int len = 1; len < last.size() && len < s[i].size(); ++len)
        {
            if (last.substr(last.size() - len) == s[i].substr(0, len))
            {
                string temp = last + s[i].substr(len);
                cnt[i] ++;
                //cout << s[i] << endl << endl;
                dfs(temp, l + s[i].size() - len);
                cnt[i] --;
                break;
            }
        }
    }
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; ++i) cin >> s[i];
    cin >> start;
    
    for (int i = 0; i < n; ++i)
    {
        if (s[i][0] == start) 
        {
            cnt[i] ++;
            dfs(s[i], s[i].size());
            cnt[i] --;
        }
    }
    
    cout << res;
}
```

