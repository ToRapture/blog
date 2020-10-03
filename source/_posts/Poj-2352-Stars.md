---
title: Poj 2352 Stars
date: 2017-09-19 17:06:00
tags:
- Algorithm
---

简单二维偏序

```CPP
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <vector>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int N = 32000 + 16;

struct Point {
    int x, y;
    Point() {}
    Point(int x, int y): x(x), y(y) {}
    bool operator < (const Point &rhs) const { return x < rhs.x || (x == rhs.x && y < rhs.y); }
    void read() { scanf("%d%d", &x, &y); ++x, ++y; }
} P[N];

struct Bit {
    int C[N];
    int n;
    void init(int n) {
        this->n = n;
        fill(C, C + n + 1, 0);
    }
    int lowbit(int x) { return x & -x; }
    void add(int x, int v) {
        for(; x <= n; x += lowbit(x)) C[x] += v;
    }
    int sum(int x) {
        int res = 0;
        for(; x; x -= lowbit(x)) res += C[x];
        return res;
    }
    int sum(int l, int r) {
        return sum(r) - sum(l - 1);
    }
} bit;

int ans[N];

int main(int argc, char **argv) {
    int n;
    while(~scanf("%d", &n)) {
        for(int i = 0; i < n; ++i) P[i].read();
        fill(ans, ans + n, 0);
        sort(P, P + n);
        bit.init(N - 1);
        for(int i = 0; i < n; ++i) {
            ++ans[bit.sum(P[i].y)];
            bit.add(P[i].y, 1);
        }
        for(int i = 0; i < n; ++i) printf("%d\n", ans[i]);
    }
    return 0;
}
```