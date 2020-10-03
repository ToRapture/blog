---
title: HDU 2069 Coin Change 生成函数
date: 2017-08-30 15:47:00
tags:
- Algorithm
---

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

typedef long long LL;
using namespace std;

const int N = 512;
LL ans[N][128], tmp[N][128];
int main(int argc, char **argv) {
    int n; while(~scanf("%d", &n)) {
        int v[] = {0, 1, 5, 10, 25, 50};
        memset(ans, 0, sizeof(ans));
        for(int i = 0; i <= 100; ++i) ans[i][i] = 1;
        memset(tmp, 0, sizeof(tmp));
        for(int i = 2; i <= 5; ++i) {
          for(int e = 0; e <= n; ++e)
		    for(int take = 0; take <= 100 && e + take * v[i] <= n; ++take)
                for(int p_take = 0; take + p_take <= 100; ++p_take) {
                tmp[e + take * v[i]][take + p_take] += ans[e][p_take];
            }
            memcpy(ans, tmp, sizeof(tmp));
            memset(tmp, 0, sizeof(tmp));
        }
        LL res = 0;
        for(int i = 0; i <= 100; ++i) res += ans[n][i];
        printf("%lld\n", res);
    }
    return 0;
}
```