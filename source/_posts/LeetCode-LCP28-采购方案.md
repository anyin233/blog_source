---
title: LeetCode LCP28.采购方案
date: 2022-02-04 11:45:11
tags: [LeetCode,LCP]

---

# 算法思路

## 暴力（TLE）

直接两重循环双指针进行遍历，每遇到一个合法方案就`ans++`

## 二分

在暴力的基础上，只枚举双指针的前一个指针，后一个指针使用二分查找得到一个最大可行的位置，方案数直接与两个指针相夹的数量相夹即可。

### 优化

考虑到存在相同报价，可以在遇到相同报价的时候直接复用之前相同报价的方案数，简单地-1即可。

# 算法实现

```c++
class Solution {
public:
    int purchasePlans(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());

        int ans = 0;
        int MOD = 1000000007;
        int pre = -1, pre_cnt = 0;
        for (int i = 0; i < nums.size(); ++i)
        {
            //cout << pre << ' ' << pre_cnt;
            if (pre >= 0 && nums[i] == nums[pre] && pre_cnt > 0) // 复用先前的方案
            {
                ans = (ans + pre_cnt) % MOD;
                pre_cnt --;
                continue;
            }

            int l = i + 1, r = nums.size() - 1, mid = (l + r + 1) >> 1;
            while(l < r)
            {
                if (nums[mid] + nums[i] > target) r = mid - 1;
                else l = mid;
                mid = (l + r + 1) >> 1;
            }
            if (nums[r] + nums[i] <= target) ans = (ans + (r - i)) % MOD;
            pre = i, pre_cnt = r - i - 1;
        }

        return ans;
    }
};
```

