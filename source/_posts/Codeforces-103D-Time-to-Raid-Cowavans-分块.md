---
title: Codeforces 103D Time to Raid Cowavans 分块
date: 2017-09-19 17:21:00
tags:
- Algorithm
---

有数组$A[N], 1\le N \le 3 \cdot 10^5$，
$P$次查询，对于每次查询给出$pos$和$k$，求$\sum\limits_{pos + m \cdot k \le N}^{} A[pos + m \cdot k]$。

把所有查询按$k$分组，$k \le \sqrt N$的组$O(N)$处理，$k \gt \sqrt N$的组在$O(\sqrt N)$时间内处理，
总复杂度为$O(\sqrt N \cdot N \ + \ (N - \sqrt N) \cdot \sqrt N) = O(2 \cdot N \sqrt N - N) = O(N \sqrt N)$。

```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int N = 300000 + 16;

struct Query {
    int st, k, id;
    LL ans;
    Query(int st, int k, int id): st(st), k(k), id(id) {}
    Query() {}
};

LL a[N], v[N];
vector<Query> Q[N];
int main(int argc, char **argv) {
    int n;
    while(~scanf("%d", &n)) {
        for(int i = 1; i <= n; ++i) scanf("%I64d", &a[i]);
        for(int i = 1; i <= n; ++i) Q[i].clear();
        int q; scanf("%d", &q);
        vector<int> have;
        for(int i = 0; i < q; ++i) {
            int st, k; scanf("%d%d", &st, &k);
            have.push_back(k);
            Q[k].push_back(Query(st, k, i));
        }
        sort(have.begin(), have.end()); have.erase(unique(have.begin(), have.end()), have.end());
        int sq = sqrt(n + 1);
        for(auto k : have) {
            if(k >= sq) {
                for(auto &query : Q[k]) {
                    int pos = query.st;
                    LL res = 0;
                    while(pos <= n) {
                        res += a[pos];
                        pos += k;
                    }
                    query.ans = res;
                }
            } else {
                fill(v, v + n + 1, 0);
                for(int i = n; i >= 1; --i) {
                    v[i] = a[i];
                    if(i + k <= n) v[i] += v[i + k];
                }
                for(auto &query : Q[k]) query.ans = v[query.st];
            }
        }
        vector<LL> ans(q);
        for(auto k : have) for(auto query : Q[k]) ans[query.id] = query.ans;
        for(auto x : ans) printf("%I64d\n", x);
    }
    return 0;
} 
```