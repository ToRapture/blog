---
title: Hungarian Algorithm
date: 2020-04-14 14:04:00
mathjax: true
tags:
- Algorithm
---

## Links
https://www.renfei.org/blog/bipartite-matching.html
https://en.wikipedia.org/wiki/Matching_(graph_theory)

## Hungarian Algorithm
Given a matching M,

* an alternating path is a path that begins with an unmatched vertex and whose edges belong alternately to the matching and not to the matching.
* an augmenting path is an alternating path that starts from and ends on free (unmatched) vertices.

Traverse all the points on the left, for each point $u$ if we can find a augmenting path $\vec{p}$ starting from $u$, then we use the edges of $\vec{p}$ to update the matches of each point on $\vec{p}$.

## Problems

[HDU 1068 Girls and Boys](http://acm.hdu.edu.cn/showproblem.php?pid=1068)

This problem is to figure out the number of maximum independent set of bipartite graph.

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 1024;

vector<int> g[MAXN];
int match[MAXN];
bool vis[MAXN];

void init(int n) {
    for (int i = 0; i < n; i++) {
        g[i].clear();
    }
    fill(match, match + n, -1);
}

bool dfs(int u) {
    for (auto v : g[u]) {
        if (vis[v]) continue;
        vis[v] = true;
        if (match[v] == -1 || dfs(match[v])) {
            match[v] = u;
            return true;
        }
    }

    return false;
}

int hungary(int n) {
    int ans = 0;
    for (int i = 0; i < n; i++) {
        fill(vis, vis + n, false);
        if (dfs(i)) ans++;
    }
    return ans / 2;
}

int main(int argc, char **argv) {
    int n;
    while (scanf("%d", &n) != EOF) {
        init(n);
        for (int i = 0; i < n; i++) {
            int u, cnt;
            scanf("%d: (%d) ", &u, &cnt);
            for (int j = 0; j < cnt; j++) {
                int v;
                scanf("%d", &v);
                g[u].push_back(v);
            }
        }
        printf("%d\n", n - hungary(n));
    }
    return 0;
}
```

[HDU 1179 Ollivanders: Makers of Fine Wands since 382 BC.](http://acm.hdu.edu.cn/showproblem.php?pid=1179)

There are 3 ways to solve the problem.

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 128;
vector<int> g[MAXN];
int match[MAXN];
bool vis[MAXN];

void init(int n, int m) {
    for (int i = 1; i <= m; i++) {
        g[i].clear();
    }
    fill(match + 1, match + 1 + n, -1);
}

void add(int u, int v) {
    g[u].push_back(v);
}

bool dfs(int u) {
    for (auto v : g[u]) {
        if (vis[v]) continue;
        vis[v] = true;
        if (match[v] == -1 || dfs(match[v])) {
            match[v] = u;
            return true;
        }
    }
    return false;
}

int hungary(int n, int m) {
    int ans = 0;
    for (int i = 1; i <= m; i++) {
        fill(vis + 1, vis + 1 + n, false);
        if (dfs(i)) ans++;
    }
    return ans;
}

int main(int argc, char **argv) {
    int n, m;
    while (scanf("%d%d", &n, &m) != EOF) {
        init(n, m);
        for (int i = 1; i <= m; i++) {
            int c;
            scanf("%d", &c);
            for (int j = 0; j < c; j++) {
                int v;
                scanf("%d", &v);
                add(i, v);
            }
        }
        printf("%d\n", hungary(n, m));
    }
    return 0;
}
```

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 128 * 2;

vector<int> g[MAXN];
int match[MAXN];
bool vis[MAXN];

void add(int u, int v) {
    g[u].push_back(v);
}

void init(int n, int m) {
    for (int i = 1; i <= n + m; i++) {
        g[i].clear();
    }
    fill(match + 1, match + 1 + n, -1);
}

bool dfs(int u) {
    for (auto v : g[u]) {
        if (vis[v]) continue;
        vis[v] = true;
        if (match[v] == -1 || dfs(match[v])) {
            match[v] = u;
            return true;
        }
    }
    return false;
}

int hungarian(int n, int m) {
    int ans = 0;
    for (int u = n + 1; u <= n + m; u++) {
        fill(vis + 1, vis + 1 + n, false);
        if (dfs(u)) ans++;
    }
    return ans;
}

int main(int argc, char **argv) {
    int n, m;
    while (scanf("%d%d", &n, &m) != EOF) {
        init(n, m);
        for (int u = n + 1; u <= n + m; u++) {
            int c;
            scanf("%d", &c);
            for (int j = 0; j < c; j++) {
                int v;
                scanf("%d", &v);
                add(u, v);
                add(v, u);
            }
        }
        printf("%d\n", hungarian(n, m));
    }
    return 0;
}
```

```cpp
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int MAXN = 128 * 2;

vector<int> g[MAXN];
int match[MAXN];
bool vis[MAXN];

void init(int n, int m) {
    for (int i = 1; i <= n + m; i++) {
        g[i].clear();
    }
    fill(match + 1, match + 1 + n + m, -1);
}

bool dfs(int u) {
    for (auto v : g[u]) {
        if (vis[v]) continue;
        vis[v] = true;
        if (match[v] == -1 || dfs(match[v])) {
            match[v] = u;
            return true;
        }
    }
    return false;
}

int hungarian(int n, int m) {
    int ans = 0;
    for (int u = 1; u <= n + m; u++) {
        fill(vis + 1, vis + 1 + n + m, false);
        if (dfs(u)) ans++;
    }
    return ans / 2;
}

void add(int u, int v) {
    g[u].push_back(v);
}

int main(int argc, char **argv) {
    int n, m;
    while (scanf("%d%d", &n, &m) != EOF) {
        init(n, m);
        for (int u = n + 1; u <= n + m; u++) {
            int c;
            scanf("%d", &c);
            for (int j = 0; j < c; j++) {
                int v;
                scanf("%d", &v);
                add(u, v);
                add(v, u);
            }
        }
        printf("%d\n", hungarian(n, m));
    }
    return 0;
}
```