---
title: HDU 2642 Stars
date: 2020-04-12 18:50:00
tags:
- Algorithm
---

[Stars](http://acm.hdu.edu.cn/showproblem.php?pid=2642)

Two dimensional binary indexed tree.

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int N = 1002;

struct Bit {
    int c[N][N];
    int n, m;

    static int lowbit(int x) {
        return x & -x;
    }

    void init(int n, int m) {
        this->n = n;
        this->m = m;
        for (int i = 0; i <= n; i++) {
            for (int j = 0; j <= m; j++) {
                c[i][j] = 0;
            }
        }
    }

    void add(int x, int y, int v) {
        for (int i = x; i <= n; i += lowbit(i)) {
            for (int j = y; j <= m; j += lowbit(j)) {
                c[i][j] += v;
            }
        }
    }

    int get(int x, int y) {
        int s = 0;
        for (int i = x; i > 0; i -= lowbit(i)) {
            for (int j = y; j > 0; j -= lowbit(j)) {
                s += c[i][j];
            }
        }
        return s;
    }
} bit;

bool star[N][N];

int main(int argc, char **argv) {
    int n;
    while (scanf("%d", &n) != EOF) {
        int m = 1001;
        bit.init(m, m);
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= m; j++) {
                star[i][j] = false;
            }
        }
        for (int i = 0; i < n; i++) {
            char op[2];
            scanf("%s", op);
            if (op[0] == 'B') {
                int x, y;
                scanf("%d%d", &x, &y);
                x++, y++;
                if (!star[x][y]) {
                    star[x][y] = true;
                    bit.add(x, y, 1);
                }
            } else if (op[0] == 'D') {
                int x, y;
                scanf("%d%d", &x, &y);
                x++, y++;
                if (star[x][y]) {
                    star[x][y] = false;
                    bit.add(x, y, -1);
                }
            } else {
                int x1, x2, y1, y2;
                scanf("%d%d%d%d", &x1, &x2, &y1, &y2);
                if (x1 > x2) {
                    swap(x1, x2);
                }
                if (y1 > y2) {
                    swap(y1, y2);
                }
                x1++, x2++, y1++, y2++;
                int res = bit.get(x2, y2) - bit.get(x1 - 1, y2) - bit.get(x2, y1 - 1) + bit.get(x1 - 1, y1 - 1);
                printf("%d\n", res);
            }
        }
    }
    return 0;
}
```