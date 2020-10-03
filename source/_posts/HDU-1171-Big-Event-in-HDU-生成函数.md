---
title: HDU 1171 Big Event in HDU 生成函数
date: 2017-08-30 15:46:00
tags:
- Algorithm
---

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

typedef long long LL;
using namespace std;
const int MAX_SUM = 250000 + 16;
const int N = 128;
LL ans[MAX_SUM], tmp[MAX_SUM];
int num[N], v[N];
int main(int argc, char **argv) {
    int n;
    while(~scanf("%d", &n) && n >= 0) {
        int tot = 0;
        for(int i = 1; i <= n; ++i) {
            scanf("%d%d", &v[i], &num[i]);
            tot += v[i] * num[i];
        }
        memset(ans, 0, sizeof(ans));
        memset(tmp, 0, sizeof(tmp));
        for(int i = 0; i <= num[1]; ++i) ans[i * v[1]] = 1;
        for(int i = 2; i <= n; ++i) {
          for(int e = 0; e <= tot; ++e) 
			  for(int take = 0; take <= num[i] && e + take * v[i] <= tot; ++take)
                tmp[e + take * v[i]] += ans[e];
            memcpy(ans, tmp, sizeof(tmp));
            memset(tmp, 0, sizeof(tmp));
        }
        int l = tot, r = 0, diff = tot;
        for(int i = 0; i <= tot; ++i) if(ans[i]) {
            int ll = i, rr = tot - i;
            int df = abs(rr - ll);
            if(ll >= rr && df < diff) {
                l = ll;
                r = rr;
                diff = df;
            }
        }
        printf("%d %d\n", l, r);
    }
    return 0;
}
```