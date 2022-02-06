---
title: LeetCode LCP33.蓄水
date: 2022-02-06 13:22:48
tags: [LeetCode,LCP]
---

# 算法思路

基于给出的操作，即每次操作可以让所有的水桶进行一次注水或者升级**一个**水桶，我们可以考虑使用贪心的思路。  
由于总共**倒水**的次数取决于需要倒水次数最多的水桶，我们可以考虑采用一个堆，每次取出需要倒水次数最多的那个水桶进行升级，从而最小化倒水的次数。通过不断遍历直到最大的倒水次数只有一次的时候即可跳出升级循环。  
另外，由于本题中存在容量为0的水缸，为了避免不必要的判断，可以考虑直接进行跳过处理。

# 算法实现
```c++
class Solution {
public:
    int storeWater(vector<int>& bucket, vector<int>& vat) {
        int cnt = 0, res = 0x3f3f3f3f; // upgrade count and result
        priority_queue<pair<int, int>, vector<pair<int, int>>, less<pair<int, int>>> heap;
        for (int i = 0; i < bucket.size(); ++i)
        {
            if (bucket[i] == 0 && vat[i] != 0) cnt ++, bucket[i] ++;
            if (bucket[i] == 0 && vat[i] == 0) continue;
            heap.push({(int)(vat[i] / bucket[i] + (vat[i] % bucket[i] > 0)), i});
        } // 预处理，防止出现为0的情况
        
        while(true && heap.size())
        {
            auto p = heap.top();
            heap.pop();
            int buk = bucket[p.second] + 1, v = vat[p.second];
            res = min(res, cnt + p.first);
            bucket[p.second] ++; // upgrade;
            cnt ++;
            if (v / buk <= 1) break; 
            heap.push({v / buk + (v % buk > 0), p.second});
        }
        return res == 0x3f3f3f3f? 0 : res;
    }
};
```
