---
title: LeetCode LCP23.魔术排列
date: 2022-02-04 11:11:35
tags: [LeetCode,LCP,模拟,贪心]

---

# 算法思路

需要枚举数字K，当数字K大于等于N的时候结果将会固定
1. 由于第一次抽出的数字必然为**偶数**，当`target`的第一位数字为奇数的时候必然无法成功
2. K必然小于`target`序列最开始递增的偶数数量
	1. 当`target`前递增的偶数为数列中全部偶数的时候，还需要注意是否会继续按照数列中的奇数顺序继续枚举
3. 由于题目性质，即抽牌必须按照洗牌后的顺序进行，故第一次计算出来的K即为得到最终正确序列必然的K 

# 算法实现

```c++
class Solution {
public:
    bool isMagic(vector<int>& target) {
        if (target[0] % 2) return false;
        int maxK = 1;
        for (; maxK < target.size() && 
        target[maxK] > target[maxK - 1] && 
        !(target[maxK] & 1) && target[maxK] == target[maxK - 1] + 2; ++maxK);
        if (target[maxK] == 1 && target[maxK - 1] == (target.size() >> 1) << 1)
        {
            maxK ++;
            for (; 
                maxK < target.size() && 
                (target[maxK] | 0) && 
                target[maxK] == target[maxK - 1] + 2; 
                ++maxK);
        }

        vector<int> temp, op, after;
        bool first = true;
        for (int i = 0; i < target.size(); ++i) op.push_back(i + 1);
        while(op.size())
        {
            for (int i = 1; i < op.size(); i += 2)
            {
                temp.push_back(op[i]);
            }
            for (int i = 0; i < op.size(); i += 2)
            {
                temp.push_back(op[i]);
            } // first step

            op = temp;
            temp.clear();
            if (first && op == target) return true;
            first = false;

            for (int i = 0; i < min(maxK, (int)op.size()); ++i) after.push_back(op[i]);
            if (op.size() > maxK) op.erase(op.begin(), op.begin() + maxK);
            else op.clear();
        }

        return after == target;
    }
};
```

> 不得不吐槽一下LeetCode的编辑器实在难用，[原题链接](https://leetcode-cn.com/problems/er94lq/)
