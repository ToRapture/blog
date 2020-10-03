---
title: Offer收割编程练习赛108
date: 2019-10-30 19:55:00
tags: Algorithm
---

https://hihocoder.com/contest/offers108/problems

## A 再买优惠
暴力或二分一下
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

struct Price {
    int a, b;
    bool operator < (const Price &rhs) const {
        return this->a < rhs.a;
    }
};

int main(int argc, char **argv) {
    int n, c;
    while (~scanf("%d%d", &n, &c)) {
        vector<Price> vec;
        for (int i = 0; i < n; ++i) {
            Price p;
            scanf("%d%d", &p.a, &p.b);
            vec.push_back(p);
        }
        sort(vec.begin(), vec.end());
        int pos = upper_bound(vec.begin(), vec.end(), Price{c}) - vec.begin();
        if (pos == n) {
            printf("%d\n", vec[pos - 1].b);
        } else {
            printf("%d %d %d\n", vec[pos - 1].b, vec[pos].a - c, vec[pos].b);
        }
    }
    return 0;
}


/**
3 40
20 10
50 25
100 50

10 10 25
*/
```

## B 树与落叶
DFS + 维护 + 二分
```cpp
#include <bits/stdc++.h>

#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int N = 1000000 + 16;

vector<int> G[N];
int max_depth, drop_time[N], total[N], drop_cnt[N];

void init(int n) {
    for (int i = 0; i <= n; ++i) {
        G[i].clear();
        total[i] = 0;
        drop_cnt[i] = 0;
    }
    max_depth = 0;
}

void dfs(int u, int fa, int depth) {
    drop_time[u] = 1;
    max_depth = max(max_depth, depth);
    for (auto v : G[u]) {
        if (v == fa) continue;
        dfs(v, u, depth + 1);
        drop_time[u] = max(drop_time[u], drop_time[v] + 1);
    }
}

int main(int argc, char **argv) {
    int n, m;
    while (~scanf("%d%d", &n, &m)) {
        init(n);
        int root = 1;
        for (int i = 1; i <= n; ++i) {
            int f;
            scanf("%d", &f);
            if (f == 0) {
                root = i;
                continue;
            }
            G[f].push_back(i);
        }
        dfs(root, -1, 1);

        int len = drop_time[root] + 1;
        for (int i = 1; i <= n; i++)
            drop_cnt[drop_time[i]]++;
        total[0] = n;
        for (int i = 1; i < len; i++) {
            total[i] = total[i - 1] - drop_cnt[i];
        }

        while (m--) {
            int d;
            scanf("%d", &d);
            int pos = lower_bound(total, total + len, d, greater<int>()) - total;
            if (pos == len) {
                printf("%d\n", len);
            } else if (pos == 0) {
                printf("%d\n", 1);
            } else {
                if (abs(total[pos - 1] - d) <= abs(total[pos] - d)) {
                    printf("%d\n", pos);
                } else {
                    printf("%d\n", pos + 1);
                }
            }
        }
    }
    return 0;
}

/**
5 2
2 0 2 3 3
4 0

1
4
*/
```

## C 树上的最短边
我是用树链剖分做的
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl
#define lrrt int L, int R, int rt
#define lson L, mid, rt << 1
#define rson mid + 1, R, rt << 1 | 1
#define iall 1, n, 1

using namespace std;
typedef long long LL;

const int N = 100000 + 16;

int tree[N << 2];
vector<int> G[N];
int n, m, ans[N];
int D[N], F[N], sz[N], son[N];
int W[N], _W[N], top[N], dfs_cnt;
void dfs(int u, int fa, int dep) {
    D[u] = dep;
    F[u] = fa;
    son[u] = 0;
    sz[u] = 1;
    for(int i = 0; i < G[u].size(); ++i) {
        int v = G[u][i];
        if(v != fa) {
            dfs(v, u, dep + 1);
            sz[u] += sz[v];
            if(son[u] == 0 || sz[v] > sz[son[u]]) son[u] = v;
        }
    }
}
void dfs_hash(int u, int tp, int fa) {
    W[u] = ++dfs_cnt;
    _W[W[u]] = u;
    top[u] = tp;
    if(son[u]) dfs_hash(son[u], tp, u);
    for(int i = 0; i < G[u].size(); ++i) {
        int v = G[u][i];
        if(v == son[u] || v == fa) continue;
        dfs_hash(v, v, u);
    }
}
void build(lrrt) {
    if(L == R) {
        tree[rt] = 0x3F3F3F3F;
    } else {
        int mid = L + R >> 1;
        build(lson);
        build(rson);
        tree[rt] = min(tree[rt << 1], tree[rt << 1 | 1]);
    }
}
void add(lrrt, int pos, int v) {
    if(L == R) {
        tree[rt] = v;
    } else {
        int mid = L + R >> 1;
        if(pos <= mid) add(lson, pos, v);
        else add(rson, pos, v);
        tree[rt] = min(tree[rt << 1], tree[rt << 1 | 1]);
    }
}
int query(lrrt, int x, int y) {
    if(x <= L && R <= y) return tree[rt];
    else {
        int mid = L + R >> 1;
        if(x > mid) return query(rson, x, y);
        else if(y <= mid) return query(lson, x, y);
        else return min(query(lson, x, y), query(rson, x, y));
    }
}
int query(int u, int v) {
    int res = 0x3F3F3F3F;
    while(top[u] != top[v]) {
        if(D[top[u]] < D[top[v]]) swap(u, v);
        res = min(res, query(iall, W[top[u]], W[u]));
        u = F[top[u]];
    }
    if(D[u] > D[v]) swap(u, v);
    if (u != v) {
        res = min(res, query(iall, W[son[u]], W[v]));
    }
    return res;
}
void add(int u, int val) {
    add(iall, W[u], val);
}
void init() {
    for(int i = 1; i <= n; ++i) G[i].clear();
    dfs_cnt = 0;
}

int pa[N], wc[N];

int main(int argc, char **argv) {
    while(~scanf("%d%d", &n, &m)) {
        init();
        int root = 1;
        for (int i = 1; i <= n; ++i) {
            scanf("%d%d", &pa[i], &wc[i]);
            if (pa[i] == 0) {
                root = i;
                continue;
            }
            G[pa[i]].push_back(i);
            G[i].push_back(pa[i]);
        }
        dfs(root, -1, 0); dfs_hash(root, root, -1);
        build(iall);
        for (int i = 1; i <= n; ++i) {
            if (wc[i] == 0)
                continue;
            add(i, wc[i]);
        }
        while (m--) {
            int u, v;
            scanf("%d%d", &u, &v);
            printf("%d\n", query(u, v));
        }
    }
    return 0;
}

/**
5 2
2 1
0 0
2 2
3 3
3 4
4 1
2 5

1
2
*/
```

## D 01匹配
线段树
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

#define lrrt int L, int R, int rt
#define lson L, mid, rt << 1
#define rson mid + 1, R, rt << 1 | 1
#define iall 1, n, 1

using namespace std;
typedef long long LL;

const int N = 300000 + 16;

struct Node {
    int ans, free_one, free_zero;
    Node() {}
    Node(int ans, int free_one, int free_zero):
        ans(ans), free_one(free_one), free_zero(free_zero) {}
    Node operator &(const Node &rhs) const {
        Node ret;
        ret.ans = this->ans + rhs.ans;
        int add = min(this->free_one, rhs.free_zero);
        ret.ans += add;
        ret.free_one = this->free_one + rhs.free_one - add;
        ret.free_zero = this->free_zero + rhs.free_zero - add;
        return ret;
    }
};

int a[N];
Node tree[N << 2];

void build(lrrt) {
    tree[rt].ans = 0;
    tree[rt].free_zero = 0;
    tree[rt].free_one = 0;
    if (L == R) {
        if (a[L]) tree[rt].free_one = 1;
        else tree[rt].free_zero = 1;
        return;
    }
    int mid = (L + R) / 2;
    build(lson);
    build(rson);
    tree[rt] = tree[rt << 1] & tree[rt << 1 | 1];
}

Node query(lrrt, int x, int y) {
    if (x <= L && R <= y) {
        return tree[rt];
    }
    int mid = (L + R) / 2;
    if (y <= mid) return query(lson, x, y);
    else if (x > mid) return query(rson, x, y);
    return query(lson, x, y) & query(rson, x, y);
}

int main(int argc, char **argv) {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++)
            scanf("%d", &a[i]);
        build(iall);
        int m;
        scanf("%d", &m);
        while (m--) {
            int l, r;
            scanf("%d%d", &l, &r);
            printf("%d\n", query(iall, l, r).ans);
        }
    }
    return 0;
}

/**
10
0 1 0 1 0 0 1 0 1 0
100
1 10

4
*/
```