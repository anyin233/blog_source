---
title: LeetCode 29.两数相除
date: 2022-02-03 22:41:41
tags: [LeetCode]

---

# 算法思路

~~笑死，直接利用右移和左移作为乘二和除以二来用，同时将变量统一采用`long long`保存，通过将被除数减去$2^n$倍的除数，累计这个$2^n$倍最终可以得到结果。~~

（上面的思路漏看了题目下面的提示，即不允许使用`long long`）

由于不允许使用`long long`，又希望尽可能获得更大的可用数字的绝对值空间（即希望可以让计算的绝对值扩大到$2^{32}$)，故最终选择所有的减法模拟的除法都使用负数进行计算。

# 算法实现

## 错误的实现（使用了64位整数）

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

## 正确的实现

```c++
class Solution {
public:
    int divide(int dividend, int divisor) {
        // 特判可能会溢出的情况
        bool fix = false;
        if (dividend == INT_MIN) 
        {
            if (divisor == -1)return INT_MAX;
            if (divisor == 1) return INT_MIN;
        }

        // 特判被除数为0
        if (dividend == 0) return 0;

        // 特判除数为1
        if (divisor == 1) return dividend;
        bool flag = (dividend >> 31) != (divisor >> 31);
        int aend = dividend > 0 ? -dividend : dividend, aor = divisor > 0 ? -divisor : divisor;

        int temp = aor, cnt = 1, ans = 0;
        while(aend <= aor)
        {
            if (aend <= temp)
            {
                aend -= temp;
                ans += cnt;
                if (temp >= (INT_MIN >> 1)) // 防溢出
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
        
        return flag ? -ans : ans;
    }
};
```

