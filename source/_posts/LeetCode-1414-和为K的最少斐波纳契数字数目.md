---
title: LeetCode 1414.和为K的最少斐波纳契数字数目
date: 2022-02-03 22:14:24
tags: [LeetCode,每日一题]

---

# 算法思路

考虑采用深度优先搜索的方式进行，由于每个斐波纳契数字均小于前一个数字的两倍（2除外），可以基于这个特性，每次找到小于当前所求总和的最大的斐波纳契数字x，然后`总和 -= x`，反复迭代之后即可得到结果。

# 算法实现

```c++
class Solution {
public:
    int f[50], n, ans;
    void init(int k){
        f[0] = 1;
        f[1] = 1;
        for (int i = 1; f[i] <= k; ++i)
        {
            f[i + 1] = f[i] + f[i - 1];
            n ++;
        }
    }
    int findMinFibonacciNumbers(int k) {
        n = 0, ans = 0;
        init(k);
        //cout << n;
        while(k)
        {
            while(f[n] > k) n--;
            k -= f[n];
            ans ++;
        }

        return ans;
    }
};
```

> [题目🔗](https://leetcode-cn.com/problems/find-the-minimum-number-of-fibonacci-numbers-whose-sum-is-k)
