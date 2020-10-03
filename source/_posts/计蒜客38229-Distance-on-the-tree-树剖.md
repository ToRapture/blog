---
title: 计蒜客38229 Distance on the tree 树剖
date: 2019-04-26 16:02:00
mathjax: true
tags:
- Algorithm
---

按照查询的边权w排序，离线add和query，注意树剖处理边权时的特殊逻辑。
```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl
#define lrrt int L, int R, int rt
#define lson L, mid, rt << 1
#define rson mid + 1, R, rt << 1 | 1
#define iall 1, n, 1
 
using namespace std;
typedef long long LL;

const int N = 500000 + 16;

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
        tree[rt] = 0;
    } else {
        int mid = L + R >> 1;
        build(lson);
        build(rson);
        tree[rt] = tree[rt << 1] + tree[rt << 1 | 1];
    }
}
void add(lrrt, int pos, int v) {
    if(L == R) {
        tree[rt] += v;
    } else {
        int mid = L + R >> 1;
        if(pos <= mid) add(lson, pos, v);
        else add(rson, pos, v);
        tree[rt] = tree[rt << 1] + tree[rt << 1 | 1];
    }
}
int query(lrrt, int x, int y) {
    if(x <= L && R <= y) return tree[rt];
    else {
        int mid = L + R >> 1;
        if(x > mid) return query(rson, x, y);
        else if(y <= mid) return query(lson, x, y);
        else return query(lson, x, y) + query(rson, x, y);
    }
}
int query(int u, int v) {
    int res = 0;
    while(top[u] != top[v]) {
        if(D[top[u]] < D[top[v]]) swap(u, v);
        res += query(iall, W[top[u]], W[u]);
        u = F[top[u]];
    }
    if(D[u] > D[v]) swap(u, v);
    if (u != v) {
        res += query(iall, W[son[u]], W[v]);
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

struct Op {
    int tp, u, v, i;
    LL w;
    bool operator < (const Op &rhs) const {
        return (this->w < rhs.w) || (this->w == rhs.w && this->tp == 0);
    }
    Op(int tp, int u, int v, LL w, int i): tp(tp), u(u), v(v), w(w), i(i) {}
};
int main(int argc, char **argv) {
    while(~scanf("%d%d", &n, &m)) {
        init();
        vector<Op> vec;
        for(int i = 1; i < n; ++i) {
            int u, v, w; scanf("%d%d%d", &u, &v, &w);
            G[u].push_back(v);
            G[v].push_back(u);
            vec.push_back(Op(0, u, v, w, 0));
        }
        for (int i = 1; i <= m; ++i) {
            int u, v, w; scanf("%d%d%d", &u, &v, &w);
            vec.push_back(Op(1, u, v, w, i));
        }
        sort(vec.begin(), vec.end());
        dfs(1, 1, 0); dfs_hash(1, 1, 1);
        build(iall);
        for (auto &op : vec) {
            if (op.tp == 1) {
                if (op.u == op.v) ans[op.i] = 0;
                else ans[op.i] = query(op.u, op.v);
            } else {
                if (D[op.u] > D[op.v]) add(op.u, 1);
                else add(op.v, 1); 
            }
        }
        for (int i = 1; i <= m; ++i)
            printf("%d\n", ans[i]);
    }
    return 0;
}
 
/**
3 3
1 3 2
2 3 7
1 3 0
1 2 4
1 2 7
5 2
1 2 1000000000
1 3 1000000000
2 4 1000000000
3 5 1000000000
2 3 1000000000
4 5 1000000000
5 7 
1 2 1000000000
1 3 1000000000
2 4 1000000000
3 5 1000000000
2 3 1000000000
4 5 1000000000
1 1 10000000000
2 2 10000000000
3 3 10000000000
4 4 10000000000
5 5 10000000000

0
1
2
2
4
*/
```