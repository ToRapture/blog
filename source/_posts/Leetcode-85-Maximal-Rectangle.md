---
title: Leetcode 85. Maximal Rectangle
date: 2018-08-31 23:14:00
tags: Algorithm
---

令$sum_{i, j}$表示从$(i, j)$开始沿着$(i - 1, j), (i - 2, j) \dots (0, j)$方向的连续的$1$的个数，
$$
sum_{i, j} = \begin{cases}
	0 & m_{i, j} \ne 1 \\
	1 + sum_{i - 1, j} & m_{i, j} = 1 \\
\end{cases}
$$
则答案为$\max \left\{(k - j + 1) \times \min\{sum_{i, l} | \ j \le l \le k  \} \ | \ 0 \le i < n, \ 0 \le j \le k < m  \right\}$。
枚举$i, j, k$可以用$O(n \times m^2)$的复杂度实现，用单调栈优化后可以$O(n \times m)$复杂度实现。

$O(n \times m^2)$实现：
```cpp
class Solution {
public:
    int maximalRectangle(vector<vector<char>> &matrix) {
        int n = matrix.size();
        if (n == 0) return 0;
        int m = matrix[0].size();
        if (m == 0) return 0;
        vector<vector<int>> sum(n, vector<int>(m));

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (matrix[i][j] == '1') {
                    sum[i][j] = 1;
                    if (i > 0) sum[i][j] += sum[i - 1][j];
                } else {
                    sum[i][j] = 0;
                }
            }
        }

        int ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                int min_sum = sum[i][j];
                for (int k = j; k < m; k++) {
                    min_sum = min(min_sum, sum[i][k]);
                    ans = max(ans, (k - j + 1) * min_sum);
                }
            }
        }
        return ans;
    }
};
```

$O(n \times m)$实现：
```cpp

class Solution {
public:
    int maximalRectangle(vector<vector<char>> &matrix) {
        int n = matrix.size();
        if (n == 0) return 0;
        int m = matrix[0].size();
        if (m == 0) return 0;
        vector<vector<int>> sum(n, vector<int>(m + 1));
        vector<int> L(m + 1);

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (matrix[i][j] == '1') {
                    sum[i][j] = 1;
                    if (i > 0) sum[i][j] += sum[i - 1][j];
                } else {
                    sum[i][j] = 0;
                }
            }
            sum[i][m] = -1;
        }

        stack<int> stk;
        int ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j <= m; j++) {
                L[j] = j;
                while (!stk.empty() && sum[i][j] < sum[i][stk.top()]) {
                    L[j] = L[stk.top()];
                    ans = max(ans, (j - L[stk.top()]) * sum[i][stk.top()]);
                    stk.pop();
                }
                stk.push(j);
            }

        }
        return ans;
    }
};
```