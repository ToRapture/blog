---
title: 计蒜客38228 Max answer 单调栈 + 线段树
date: 2019-04-24 17:15:00
mathjax: true
tags:
- Algorithm
---

[Max answer](https://nanti.jisuanke.com/t/38228)
与[POJ 2796 Feel Good](http://poj.org/problem?id=2796)类似，但是这个题有负数，需要特殊处理一下

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

#define iall 1, n, 1
#define lrrt int l, int r, int rt
#define lson l, mid, rt << 1
#define rson mid + 1, r, rt << 1 | 1

const int MAXN = 500000 + 16;
const LL INF = 0x7F7F7F7F7F7F7F7F;
LL a[MAXN], sum[MAXN];
LL max_tree[MAXN << 2], min_tree[MAXN << 2];
int STK[MAXN], top;
int L[MAXN], R[MAXN];
int n;

void build(lrrt) {
    max_tree[rt] = min_tree[rt] = 0;
    if (l == r) {
        return;
    }
    int mid = (l + r) >> 1;
    build(lson);
    build(rson);
}

void add(lrrt, int x, int v) {
    if (l == r) {
        max_tree[rt] = min_tree[rt] = v;
        return;
    }
    int mid = (l + r) >> 1;
    if (x <= mid) add(lson, x, v);
    else add(rson, x, v);
    max_tree[rt] = max(max_tree[rt << 1], max_tree[rt << 1 | 1]);
    min_tree[rt] = min(min_tree[rt << 1], min_tree[rt << 1 | 1]);
}

LL query_min(lrrt, int x, int y) {
    if (x <= l && r <= y) {
        return min_tree[rt];
    }
    int mid = (l + r) >> 1;
    if (x > mid) return query_min(rson, x, y);
    else if (y <= mid) return query_min(lson, x, y);
    else return min(query_min(lson, x, y), query_min(rson, x, y));
}

LL query_max(lrrt, int x, int y) {
    if (x <= l && r <= y) {
        return max_tree[rt];
    }
    int mid = (l + r) >> 1;
    if (x > mid) return query_max(rson, x, y);
    else if (y <= mid) return query_max(lson, x, y);
    else return max(query_max(lson, x, y), query_max(rson, x, y));
}

LL get_max(int x, int y) {
    if (y == 0)
        return 0;
    if (x == 0) {
        return max(LL(0), query_max(iall, 1, y));
    }
    return query_max(iall, x, y); 
}

void get_lr() {
    top = 0;
    for (int i = 1; i <= n; ++i) {
        L[i] = i;
        while (top && a[i] < a[STK[top]]) {
            L[i] = L[STK[top]];
            R[STK[top]] = i - 1;
            --top;
        }
        STK[++top] = i;
    }
    while (top) {
        R[STK[top--]] = n;
    }
}

int main(int argc, char **argv) {
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; ++i) scanf("%lld", &a[i]); a[n + 1] = -INF;
        for (int i = 1; i <= n; ++i) sum[i] = sum[i - 1] + a[i];
        get_lr();
        build(iall);
        for (int i = 1; i <= n; ++i) add(iall, i, sum[i]);
        LL ans = -INF;
        for (int i = 1; i <= n; ++i) {
            if (a[i] >= 0) {
                LL tmp = a[i] * (sum[R[i]] - sum[L[i] - 1]);
                ans = max(ans, tmp);
            } else {
                LL sr = query_min(iall, i, R[i]);
                LL sl = get_max(L[i] - 1, i - 1); 
                LL tmp = a[i] * (sr - sl);
                ans = max(ans, tmp);
            }
        }
        printf("%lld\n", ans);
    }
}

/**
5
1 2 3 4 5
5
-1 -2 -3 -4 -5
7
1 2 3 -1 -2 -3 8
7
1 2 3 -1 -2 -3 1
9
1 2 3 -1 -2 -3 8 -100 100

*/
```