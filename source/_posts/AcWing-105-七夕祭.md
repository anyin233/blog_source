---
title: AcWing 105. 七夕祭
date: 2022-01-09 15:44:08
tags: [算法学习,AcWing,排序]

---

# 算法思路

考虑到本题的实际操作机制，行内相邻交换不会导致列分布变化、列内交换不会导致行分布变化，根据这点我们可以将本题拆分为分别对行和列做循环交换操作。

假设一个环形结构$[a_1, a_2 ... a_n]$，需要将其中所有的数字通过相邻数字间的交换变为$a$，假设从$a_i \to a_{i + 1}$传递$x_i$个数字($x_i$可正可负，且$a_n\to a_1$为$x_n$)则有

$$a_1 + x_n - x_1 = a \\ a_2 + x_1 - x_2 = a \\ ... \\a_n + x_{n - 1} - x_n = a$$

整理得

$$x_1 - x_n = a_1 - a \\ x_2 - x_1 = a_2 - a \\ ... \\ x_n - x_{n - 1} = a_n - a$$

由于希望求得最小的操作次数，即题目希望求得最小的$|x_1| + |x_2| + ... +|x_n|$则有

$$x_1 = x_1 \\ x_2 = x_1 - (a - a_2) \\ x_3 = x_2 - (a - a_3) = x_1 - (2a - a_2 - a_3) \\ ... \\ x_n = x_{n - 1} - (a - a_n) = ... = x_1 - ((n - 1)a-a_2-a_3 ... -a_n)$$

即最终上面的式子被转化为了$|x_1| + |x_1 - (a-a_2)| ...+|x_1 - [(n - 1)a - a_2 - a_3 ... -a_n]|$最终变为了货仓选址问题。

<!---more--->

# 代码实现

```c++
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n, m, t;
int r[N], c[N];

long long compute(int a[], int avg, int cnt)
{
    int buf[N];
    for (int i = 2; i <= cnt; ++i)
    {
        buf[i] = avg - a[i] + buf[i - 1]; // buf[1] = a - a_2 buf[2] = 2a - a_2 - a_3 ...
    }
    sort(buf + 1, buf + cnt + 1);
    int mid = (1 + cnt) >> 1;
    long long res = 0;
    for (int i = 1; i <= cnt; ++i)
    {
        res += abs(buf[i] - buf[mid]);
    }
    
    return res;
}

int main() 
{
    cin >> n >> m >> t;
    for (int i = 0; i < t; ++i)
    {
        int x, y;
        cin >> x >> y;
        r[x] ++;
        c[y] ++;
    }
    
    int col_res = 0, row_res = 0;
    int col_avg = 0, row_avg = 0;
    
    if (t % m && t % n) cout << "impossible";
    else if (t % m)
    {
        cout << "row ";
        cout << compute(r, t / n, n);
        
    }
    else if (t % n)
    {
        cout << "column ";
        cout << compute(c, t / m, m);
    }
    else 
    {
        cout << "both ";
        cout << compute(r, t / n, n) + compute(c, t / m, m);
    }
}
```

