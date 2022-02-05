---
title: LeetCode LCP30.魔塔游戏
date: 2022-02-05 16:06:04
tags: [LeetCode,LCP,优先队列]

---

# 算法思路

在本题中每次只能将一个负数移动到数列的最后，我们可以考虑维护一个大根堆保存所有的负数，优先移动所有较大的负数可以使得结果最优。

# 算法实现

```c++
class Solution {
public:
    int magicTower(vector<int>& nums) {
        long long sum = 0;
        for (int i = 0; i < nums.size(); ++i) sum += nums[i];
        if (sum < 0) return -1;
        
        sum = 0;
        int cnt = 0;
        priority_queue<int, vector<int>, greater<int>> heap;
        for (int i = 0; i < nums.size(); ++i)
        {
            sum += nums[i];
            if (nums[i] < 0) heap.push(nums[i]);
            while (sum < 0 && heap.size())
            {
                cnt ++;
                nums.push_back(heap.top());
                sum -= heap.top();
                //cout << sum << ' ' << heap.top()<< endl;
                heap.pop();
            }
        }

        return cnt;
    }
};
```

