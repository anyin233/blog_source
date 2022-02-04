---
title: LeetCode 1725.可以形成最大正方形的矩形数目
date: 2022-02-04 10:26:53
tags: [LeetCode,每日一题]

---

# 算法思路

还能有什么思路，每个矩形既然只能用于切割一个正方形，那么自然能够切割出最大的边长自然就是短边长度。

> 本题使用索引的效率比迭代器更高

# 算法实现

```c++
class Solution {
public:
    int countGoodRectangles(vector<vector<int>>& rectangles) {
        int maxLen = 0;
        int cnt = 0;

        for (auto v : rectangles)
        {
            int temp = min(v[0], v[1]);
            if (temp > maxLen)
            {
                maxLen = temp;
                cnt = 1;
            }
            else if (temp == maxLen)
            {
                cnt ++;
            }
        }

        return cnt;
    }
};
```

