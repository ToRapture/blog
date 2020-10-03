---
title: HDU 6203 ping ping ping LCA + 贪心
date: 2017-09-19 17:05:00
tags:
- Algorithm
---

有一个$N+1$个结点的树$T$。
现在有$Q$个信息，每条信息为$q:(u,v)$，表示$u$到$v$的路径不通。
问树$T$至少被破坏多少个节点才能达到当前这种局面。


```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

#define mt(a, b) memset(a, b, sizeof(a))

struct LCA { ///最近公共祖先 build O(V*logV) query O(1)
    typedef int typec; ///边权的类型
    static const int MV = 100000 + 16; ///点的个数
    static const int MR = MV << 1; ///rmq 的个数

    int Index, H[MV], dex[MR], a[MR], b[MR], F[MR], ra[MR][20], rb[MR][20];
    typec dist[MV];
    struct G {
        struct E {
            int v, next;
            typec w;
        } e[MV << 1];
        int le, head[MV];
        void init(int n) {
            le = 0;
            for(int i = 0; i <= n; i++) head[i] = -1;
        }
        void add(int u, int v, typec w) {
            e[le].v = v;
            e[le].w = w;
            e[le].next = head[u];
            head[u] = le++;
        }
    } g;
    void init_dex() {
        dex[0] = -1;
        for(int i = 1; i < MR; i++) {
            dex[i] = dex[i >> 1] + 1;
        }
    }
    void dfs(int u, int fa, int level) {
        a[Index] = u;
        b[Index++] = level;
        F[u] = fa;
        for(int i = g.head[u]; ~i; i = g.e[i].next) {
            int v = g.e[i].v;
            if(v == fa) continue;
            dist[v] = dist[u] + g.e[i].w;
            dfs(v, u, level + 1);
            a[Index] = u;
            b[Index++] = level;
        }
    }

    LCA() {
        init_dex();
    };
    void init(int n) {
        g.init(n);
    }

    void add(int u, int v, typec w) {
        g.add(u, v, w);
        g.add(v, u, w);
    }
    void build(int root) { ///传入树根
        Index = 0;
        dist[root] = 0;
        dfs(root, -1, 0);
        mt(H, -1);
        mt(rb, 0x3f);
        for(int i = 0; i < Index; i++) {
            int tmp = a[i];
            if(H[tmp] == -1) H[tmp] = i;
            rb[i][0] = b[i];
            ra[i][0] = a[i];
        }
        for(int i = 1; (1 << i) <= Index; i++) {
            for(int j = 0; j + (1 << i) < Index; j++) {
                int next = j + (1 << (i - 1));
                if(rb[j][i - 1] <= rb[next][i - 1]) {
                    rb[j][i] = rb[j][i - 1];
                    ra[j][i] = ra[j][i - 1];
                    continue;
                }
                rb[j][i] = rb[next][i - 1];
                ra[j][i] = ra[next][i - 1];
            }
        }
    }
    int getlca(int l, int r) { ///查最近公共祖先
        l = H[l];
        r = H[r];
        if(l > r) swap(l, r);
        int p = dex[r - l + 1];
        r -= (1 << p) - 1;
        return rb[l][p] <= rb[r][p] ? ra[l][p] : ra[r][p];
    }
    typec getdist(int id) { ///查根到点最短距离
        return dist[id];
    }
    typec getdist(int l, int r) { ///查两点最短距离
        return dist[l] + dist[r] - 2 * dist[getlca(l, r)];
    }
} yoshiko;

int n, q;

struct Query {
    int u, v, lca, d;
    Query(int u, int v, int lca, int d): u(u), v(v), lca(lca), d(d) {}
    Query() {}
    bool operator < (const Query &rhs) const {
        return d > rhs.d;
    }
};

bool vis[100000 + 16];

void dfs(int u) {
    vis[u] = true;
    for(int i = yoshiko.g.head[u]; ~i; i = yoshiko.g.e[i].next) {
        int v = yoshiko.g.e[i].v;
        if(v != yoshiko.F[u] && !vis[v]) dfs(v);
    }
}

int main(int argc, char **argv) {
    while(~scanf("%d", &n)) {
        ++n; yoshiko.init(n);
        for(int i = 1; i < n; ++i) {
            int u, v; scanf("%d%d", &u, &v);
            yoshiko.add(u, v, 1);
        }
        yoshiko.build(0);
        vector<Query> Q;
        scanf("%d", &q);
        while(q--) {
            int u, v; scanf("%d%d", &u, &v);
            int lca = yoshiko.getlca(u, v);
            int dist = yoshiko.getdist(lca);
            Q.push_back(Query(u, v, lca, dist));
        }

        sort(Q.begin(), Q.end());

        int ans = 0;
        memset(vis, false, sizeof(vis));
        for(auto &query : Q) {
            if(!vis[query.u] && !vis[query.v]) {
                ++ans;
                dfs(query.lca);
            }
        }
        printf("%d\n", ans);
    }
    return 0;
}
```