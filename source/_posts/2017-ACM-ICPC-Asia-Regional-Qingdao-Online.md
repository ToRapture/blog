---
title: 2017 ACM/ICPC Asia Regional Qingdao Online
date: 2017-09-17 20:28:00
mathjax: true
tags:
- Algorithm
---

[1001 Apple](http://acm.hdu.edu.cn/showproblem.php?pid=6206)
给了四个点$A,B,C,P$，问点$P$是否在$\Delta_{ABC}$的外接圆外。
用C++被卡了精度，用Java过的。

```Java
import java.math.BigDecimal;
import java.util.Scanner;

/**
 * Created by ToRapture on 2017/9/17.
 */
public class Main {

    static BigDecimal dist(Point a, Point b) {
        return a.x.subtract(b.x).multiply(a.x.subtract(b.x)).add(a.y.subtract(b.y).multiply(a.y.subtract(b.y)));
    }
    static BigDecimal circumscir(Point a, Point b, Point c, Point res) {
        BigDecimal bx = b.x.subtract(a.x);
        BigDecimal by = b.y.subtract(a.y);

        BigDecimal cx = c.x.subtract(a.x);
        BigDecimal cy = c.y.subtract(a.y);

        BigDecimal d = BigDecimal.valueOf(2L).multiply(bx.multiply(cy).subtract(by.multiply(cx)));

        BigDecimal x = cy.multiply( bx.multiply(bx).add(by.multiply(by))).subtract(by.multiply( cx.multiply(cx).add(cy.multiply(cy))))
                .divide(d).add(a.x);
        BigDecimal y = bx.multiply( cx.multiply(cx).add(cy.multiply(cy))).subtract(cx.multiply( bx.multiply(bx).add(by.multiply(by))))
                .divide(d).add(a.y);

        res.x = x;
        res.y = y;
        BigDecimal r = dist(a, res);
        return r;
    }
    static int dcmp(BigDecimal x) {
        BigDecimal eps = new BigDecimal("0.000000000000001");
        if(x.compareTo(eps) > 0) return 1;
        else if(x.compareTo(eps.multiply(new BigDecimal("-1"))) < 0) return -1;
        else return 0;

    }
    static boolean fuck(Point a, Point b, Point c, Point p) {
        if(p.equals(a) || p.equals(b) || p.equals(c)) return false;
        Point o = new Point();
        BigDecimal r = circumscir(a, b, c, o);

        if(dcmp(dist(p, o).subtract(r)) <= 0) return false;
        else return true;
    }

    static public void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        int T = scan.nextInt();
        while(--T >= 0) {
            Point a = new Point();
            Point b = new Point();
            Point c = new Point();
            Point p = new Point();
            a.read(scan);
            b.read(scan);
            c.read(scan);
            p.read(scan);
            if(fuck(a, b, c, p)) System.out.println("Accepted");
            else System.out.println("Rejected");

        }
        scan.close();
    }
}

class Point {
    BigDecimal x, y;
    Point() {}
    Point(BigDecimal x, BigDecimal y) {
        this.x = x;
        this.y = y;
    }
    public void read(Scanner scan) {
        x = scan.nextBigDecimal();
        y = scan.nextBigDecimal();
    }
}
```

[1003 The Dominator of Strings](http://acm.hdu.edu.cn/showproblem.php?pid=6208)
Hash加乱搞过的，据说暴力strstr也能过。

```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;
struct HASH {
    typedef LL typec;
    static const int MaxN = 100000 + 16;
    static const LL key = 137;

    typec H[MaxN], xp[MaxN];

    void init(const char s[], int len) {
        H[len] = 0;
        for(int i = len - 1; i >= 0; --i) {
            H[i] = H[i + 1] * key + s[i];
        }
        xp[0] = 1;
        for(int i = 1; i <= len; ++i) {
            xp[i] = xp[i - 1] * key;
        }
    }
    typec get(int pos, int len) {
        return H[pos] - H[pos + len] * xp[len];
    }
} yoshiko;
const int N = 100000 + 16;
char buf[N];
LL H[N];
vector<int> F[256];
char fst[N];
int L[N];
int main(int argc, char **argv) {
    int T; scanf("%d", &T);
    while(T--) {
        int n; scanf("%d", &n);
        string str;
        int select = -1;
        for(int i = 0; i < n; ++i) {
            scanf("%s", buf);
            int l = strlen(buf);
            fst[i] = buf[0];
            L[i] = l;
            if(str.size() < l) {
                str = buf;
                select = i;
            }
            LL h = 0;
            for(int i = l - 1; i >= 0; --i) h = h * 137LL + buf[i];
            H[i] = h;
        }
        yoshiko.init(str.c_str(), str.size());
        for(char c = 'a'; c <= 'z'; ++c) F[c].clear();
        for(int i = 0; i < str.size(); ++i) F[str[i]].push_back(i);
        bool ok = true;

        for(int i = 0; i < n && ok; ++i) {
            if(i == select) continue;
            ok = false;
            for(auto p : F[fst[i]]) {
                if(p + L[i] <= str.size() && yoshiko.get(p, L[i]) == H[i]) {
                    ok = true;
                    break;
                }

            }
        }

        if(ok) puts(str.c_str());
        else puts("No");

    }
    return 0;
}
```


[1008 Chinese Zodiac](http://acm.hdu.edu.cn/showproblem.php?pid=6213)
签到题。

```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

int main(int argc, char **argv) {
    map<string, int> mp;
    mp["rat"] = 0;
    mp["ox"] = 1;
    mp["tiger"] = 2;
    mp["rabbit"] = 3;
    mp["dragon"] = 4;
    mp["snake"] = 5;
    mp["horse"] = 6;
    mp["sheep"] = 7;
    mp["monkey"] = 8;
    mp["rooster"] = 9;
    mp["dog"] = 10;
    mp["pig"] = 11;

    map<int, string> pm;
    for(auto it : mp) pm[it.second] = it.first;

    int T; cin >> T;
    while(T--) {
        string a, b;
        cin >> a >> b;
        int c = 0;
        if(a == b) c = 12;
        else {
            int pos = mp[a];
            while(pos != mp[b]) {
                ++c;
                pos = pos + 1;
                pos %= 12;
            }
        }
        cout << c << endl;
    }
    return 0;
}

```

[1009 Smallest Minimum Cut](http://acm.hdu.edu.cn/showproblem.php?pid=6214)
求最小割中割边数最小的割的边数。

```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;
struct Dinic {  //时间复杂度O(min(n * sqrt(n), sqrt(m)) * m)
    static const int maxn = 3000 + 16;
    static const int INF = 0x7F7F7F7F;
    struct Edge {
        int from, to;
        LL cap, flow;
        Edge(int from, int to, LL cap, LL flow):
            from(from), to(to), cap(cap), flow(flow) {}
        Edge() {}
    };
    vector<Edge> edges;
    vector<int> G[maxn];
    bool vis[maxn];
    int d[maxn], cur[maxn];

    int n, m, s, t; //n与m不需要特殊赋值，s与t在调用主过程的时候传就可以

    void init(int n = maxn - 1) {
        this->n = n;
        for(int i = 0; i <= n; ++i) G[i].clear();
        edges.clear();
    }

    void add_edge(int from, int to, LL cap) {
        edges.push_back(Edge(from, to, cap, 0));
        edges.push_back(Edge(to, from, 0, 0));
        m = edges.size();
        G[from].push_back(m - 2);
        G[to].push_back(m - 1);
    }

    bool BFS() {
        memset(vis, false, sizeof(vis));
        queue<int> Q;
        Q.push(s);
        d[s] = 0;
        vis[s] = true;
        while(!Q.empty()) {
            int x = Q.front(); Q.pop();
            for(int i = 0; i < G[x].size(); ++i) {
                Edge &e = edges[G[x][i]];
                if(!vis[e.to] && e.cap > e.flow) {
                    vis[e.to] = true;
                    d[e.to] = d[x] + 1;
                    Q.push(e.to);
                }
            }
        }
        return vis[t];
    }

    LL DFS(int x, LL a) {
        if(x == t || a == 0) return a;
        LL flow = 0, f;
        for(int &i = cur[x]; i < G[x].size(); ++i) {
            Edge &e = edges[G[x][i]];
            if(d[x] + 1 == d[e.to] && (f = DFS(e.to, min(a, e.cap - e.flow))) > 0) {
                e.flow += f;
                edges[G[x][i] ^ 1].flow -= f;
                flow += f;
                a -= f;
                if(a == 0) break;
            }
        }
        return flow;
    }
    LL MaxFlow(int s, int t) {
        this->s = s; this->t = t;
        LL flow = 0;
        while(BFS()) {
            memset(cur, 0, sizeof(cur));
            flow += DFS(s, INF);
        }
        return flow;
    }
} Dinic;

const LL BIG = 10000000LL;

int main(int argc, char **argv) {
    int T; scanf("%d", &T);
    while(T--) {
        int n, m; scanf("%d%d", &n, &m);
        int s, t; scanf("%d%d", &s, &t);
        Dinic.init(n);
        while(m--) {
            int u, v, c; scanf("%d%d%d", &u, &v, &c);
            Dinic.add_edge(u, v, c * BIG + 1);
        }
        printf("%lld\n", Dinic.MaxFlow(s, t) % BIG);
    }
    return 0;
}
```

[1011 A Cubic number and A Cubic Number](http://acm.hdu.edu.cn/showproblem.php?pid=6216)
好像也是签到题。
```CPP
#include <bits/stdc++.h>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;

typedef long long LL;

int main(int argc, char **argv) {
    int T; cin >> T;
    while(T--) {
        double d; cin >> d;
        double p = powf(d, 1.0 / 3);
        LL s = p - 1;
        if(s < 0) s = 0;
        bool fuck = false;
        for(LL i = s; ; i++) {
            LL x = i * i * i;
            LL y = (i + 1) * (i + 1) * (i + 1);
            if(y - x == d) {
                puts("YES");
                fuck = true; break;
            } else if(y - x > d) break;
        }
        if(!fuck) puts("NO");
    }
    return 0;
}
```