---
title: Poj 2796 Feel Good 单调栈
date: 2017-08-30 15:13:00
tags:
- Algorithm
---

令$sum(l, r) = \sum\limits_{i \in [l, r]} a_i$，求$\max\{\min_{i=l}^ra_i \times sum(l, r) \  |\ 1 \le l \le r \le n \}$。
对于每一个$a_i$，用单调栈维护出一个长度最大的区间$[L_i, R_i]$，使得$\min_{j=L_i}^{R_i} a_j = a_i$，答案转换为$\max\{sum(L_i, R_i) \times a_i \ | \ i \in [1, n]\}$。

证明
=======
### 问题等价
$ans(l, r) = \min_{i=l}^r \times sum(l,r)$
对于每一个$[l, r]$，有唯一二元组$(i, a_i)$满足$a_i=\min_{j=l}^r$且下标$i$最小。
令$[L,R]$为包含这个二元组的长度最大的区间，则有$a_i=\min_{j=L}^{R}$。
所以$ans(L,R)=\min_{i=L}^{R}a_i \times sum(L,R)$，又因$sum(l,r) \le sum(L,R)$，则有$ans(l,r) \le ans(L,R)$。
而一个这样的$[L,R]$对应多个$[l,r]$，每一个$[L_i,R_i]$又与$(i, a_i)$一一对应，答案即为$\max\{sum(L_i, R_i) \times a_i \ | \ i \in [1, n]\}$。

### 单调栈维护
对于每一个$a_i$，使用递增单调栈$STK$处理出$[L_i,R_i]$。
每考虑到一个元素$a_i$时，$L_i$在$a_i$入栈时被处理好，$R_i$在$a_i$出栈时处理。
若$a_i > a_j, i < j$，根据$L$的定义，有$L_j \le L_i$。
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <map>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int N = 100000 + 16;

LL a[N], sum[N];
int STK[N], top;
int L[N];
int main(int argc, char **argv) {
    int n;
    while(~scanf("%d", &n)) {
        for(int i = 1; i <= n; ++i) scanf("%lld", &a[i]); a[++n] = -1;
        for(int i = 1; i <= n; ++i) sum[i] = sum[i - 1] + a[i];

        pair<LL, pair<int, int>> ans = make_pair(-1, make_pair(-1, -1));
        top = 0;
        for(int i = 1; i <= n; ++i) {
            if(top == 0 || a[i] >= a[STK[top]]) {
                L[i] = i;
                STK[++top] = i;
            } else {
                while(top && a[i] < a[STK[top]]) {
                    int x = STK[top--];
                    int l = L[x], r = i - 1;
                    LL res = (sum[r] - sum[l - 1]) * a[x];
                    ans = max(ans, make_pair(res, make_pair(l, r)));
                    L[i] = L[x];
                }
                STK[++top] = i;
            }
        }
        printf("%lld\n", ans.first);
        printf("%d %d\n", ans.second.first, ans.second.second);
    }
    return 0;
}
```