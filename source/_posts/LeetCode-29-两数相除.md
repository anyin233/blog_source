---
title: LeetCode 29.两数相除
date: 2022-02-03 22:41:41
tags: [LeetCode]

---

# 算法思路

笑死，直接利用右移和左移作为乘二和除以二来用，同时将变量统一采用`long long`保存，通过将被除数减去$2^n$倍的除数，累计这个$2^n$倍最终可以得到结果。

# 算法实现

```c++
class Solution {
public:
    int divide(int dividend, int divisor) {
        if (divisor == 1) return dividend;
        bool flag = (dividend >> 31) != (divisor >> 31);
        long long aend = abs(dividend), aor = abs(divisor);

        long long temp = aor, cnt = 1, ans = 0;
        while(aend >= aor)
        {
            if (aend >= temp)
            {
                aend -= temp;
                ans += cnt;
                if (temp <= 1073741824)
                {
                    temp += temp;
                    cnt += cnt;
                }
            }
            else
            {
                temp >>= 1;
                cnt >>= 1;
            }
        }
        if (ans > 2147483647) ans = 2147483647;
        return flag ? -ans : ans;
    }
};
```

