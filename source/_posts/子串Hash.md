---
title: 子串Hash
date: 2019-10-31 14:52:00
tags:
- Algorithm
---

## 模板
```cpp
const int N = 1000000 + 16;
struct Hash {
    const unsigned long long KEY = 137;
    unsigned long long h[N], p[N];
    int len;
    void init(const int a[], int len) {
        this->len = len;
        p[0] = 1;
        for (int i = 1; i <= len; i++)
            p[i] = p[i - 1] * KEY;
        h[len] = 0;
        for (int i = len - 1; i >= 0; i--)
            h[i] = h[i + 1] * KEY + a[i];
    }
    unsigned long long get(int l, int r) {
        return h[l] - h[r + 1] * p[r - l + 1];
    }
    unsigned long long get_for_len(int l, int len) {
        return h[l] - h[l + len] * p[len];
    }
};
```
`get`方法中$0 \le l \le r < len(a)$，`get_for_len`方法中$l \in [0, len(a))，len \in [1, len(a) - l]$。

## 原理

$p_i = k^i$
$h_i = \sum\limits_{j \in [i,\ len(a))} a_j \cdot k^{j - i}$
$get(l, r) = h_l - h_{r+1} \cdot k^{r - l + 1}$
$= \sum\limits_{j \in [l, len(a))} a_j \cdot k^{j - l} - \sum\limits_{j \in [r+1, len(a))} a_j \cdot k^{j-r-1} \cdot k^{r-l+1}$
$= \sum\limits_{j \in [l, len(a))} a_j \cdot k^{j - l} - \sum\limits_{j \in [r+1, len(a))} a_j \cdot k^{j-l}$
$= \sum\limits_{j \in [l, r+1)} a_j \cdot k^{j - l} + \sum\limits_{j \in [r+1, len(a))} a_j \cdot k^{j - l} - \sum\limits_{j \in [r+1, len(a))} a_j \cdot k^{j-l}$
$= \sum\limits_{j \in [l, r+1)} a_j \cdot k^{j - l}$
$= \sum\limits_{j \in [l, r]} a_j \cdot k^{j - l}$
在实现的时候用到了unsigned long long的自然溢出，相当于$\mod 2^{64}$。

## 例题
[HDU 1711 Number Sequence](http://acm.hdu.edu.cn/showproblem.php?pid=1711)
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;

const int N = 1000000 + 16;

struct Hash {
    const unsigned long long KEY = 137;
    unsigned long long h[N], p[N];
    int len;
    void init(const int a[], int len) {
        this->len = len;
        p[0] = 1;
        for (int i = 1; i <= len; i++)
            p[i] = p[i - 1] * KEY;
        h[len] = 0;
        for (int i = len - 1; i >= 0; i--)
            h[i] = h[i + 1] * KEY + a[i];
    }
    unsigned long long get(int l, int r) {
        return h[l] - h[r + 1] * p[r - l + 1];
    }
    unsigned long long get_for_len(int l, int len) {
        return h[l] - h[l + len] * p[len];
    }

} hp, hs;

int a[N];

int get_ans() {
    for (int i = 0; i + hs.len - 1 < hp.len; i++)
        if (hs.get_for_len(0, hs.len) == hp.get_for_len(i, hs.len))
            return i + 1;
    return -1;
}

int main(int argc, char **argv) {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, m;
        scanf("%d%d", &n, &m);
        for (int i = 0; i < n; i++)
            scanf("%d", &a[i]);
        hp.init(a, n);
        for (int i = 0; i < m; i++)
            scanf("%d", &a[i]);
        hs.init(a, m);
        printf("%d\n", get_ans());
    }
    return 0;
}

/**
2
13 5
1 2 1 2 3 1 2 3 1 3 2 1 2
1 2 3 1 3
13 5
1 2 1 2 3 1 2 3 1 3 2 1 2
1 2 3 2 1

6
-1
*/
```