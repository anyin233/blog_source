---
title: AcWing 167.木棒
date: 2022-02-03 20:10:28
tags: [AcWing,算法学习,DFS,剪枝]

---

# 算法思路

本题需要考虑几种剪枝方式

## 减少枚举方案

同样从大到小依次进行枚举，由于最终答案中木棍长度固定，从更长的小棍开始枚举可以减少产生方案的数量。

## 可行性剪枝

对于**单根木棍无法整除所有小木棍长度之和**的情况和**加入某根小木棍后总长度超过当前枚举的总长度**的情况均可以直接进行剪枝。

## 移除冗余方案

1. 由于本题中只需要求出总和，故基于组合数进行枚举
2. 当某个长度的小木棍无法用于构造答案的时候，直接跳过所有与其等长的小木棍
3. 当某个小木棍为当前大木棍的第一根木棍的时候，只要其无法构造答案那么该总长度必然无解
4. 当某个小木棍为当前大木棍的最后一根木棍的时候，只要其无法构造答案那么该总长度必然无解

### 对于剪枝方案3的证明

假定构造当前选定的小木棍为n，其作为x的第一根小木棍构造大木棍，**但是最终该总长度不可行**。假设当前方案可行，即存在一个大木棍y，使得n可以作为第一根木棍，由于本题中大木棍无序，交换x和y，最终可以使得n作为x的第一根木棍，与前面矛盾，故作为剪枝方案。

### 对于剪枝方案4的证明

若小木棍n的长度作为大木棍x的最后一根木棍长度，**但是此时该总长度不可行**。可以知道如果**总长度可行**的情况下，x的最后部分必然可以由多个总长度与n相同的木棍构成，此时n实际上可以和这些小木棍实现等价替换，与前面矛盾，故同样可以作为剪枝方案。



# 算法实现

```c++
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 70;
int n, w[N], length, sum;
bool st[N];


bool dfs(int u, int s, int start) // 当前已经构造了u根木棍，当前正在构造的木棍长度为s，下一步从start开始(剪枝)
{
    if (u * length == sum) return true;
    if (s == length) return dfs(u + 1, 0, 0);
    
    for (int i = start; i < n; ++i)
    {
        if (st[i]) continue;
        if (s + w[i] > length) continue;
        
        st[i] = true;
        if (dfs(u, s + w[i], i + 1)) return true;
        st[i] = false;
        
        if (!s) return false; // 冗余方案剪枝方案3，采用w[i]作为第一根木棍但是全局失败，则剩余方案均无序进行枚举
        if (s + w[i] == length) return false; // 冗余方案剪枝方案4，即采用w[i]作为最后一根失败，但是w[i]可以作为最后一根
        
        int j = i;
        while(j < n && w[j] == w[i]) j ++;
        i = j - 1;
    }
    
    return false;
}

int main()
{
    while (cin >> n, n != 0)
    {
        sum = 0;
        memset(st, false, sizeof st);
        for (int i = 0; i < n; ++i) 
        {
            cin >> w[i];
            sum += w[i];
        }
        sort(w, w + n);
        reverse(w, w + n);
        
        length = w[n - 1];
        while(true)
        {
            if (sum % length == 0 && dfs(0, 0, 0))
            {
                cout << length << endl;
                break;
            }
            length ++;
        }
    }
}
```

