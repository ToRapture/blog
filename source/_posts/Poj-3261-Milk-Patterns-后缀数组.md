---
title: Poj 3261 Milk Patterns 后缀数组
date: 2017-09-17 20:05:00
mathjax: true
tags:
- Algorithm
---

求至少可重叠重复$K$次的最长子串的长度。
解法为二分长度后对height数组分组。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <iostream>
#include <set>
#define DBG(x) cerr << #x << " = " << x << endl

using namespace std;
typedef long long LL;

const int maxn = 2000000 + 16;
/*********************************************
*********************
** 后缀数组 Suffix Array
** INIT：solver.call_fun(char* s);
** CALL: solver.lcp(int i,int j); //后缀 i 与后缀 j 的最
长公共前缀
** SP_USE: solver.LCS(char *s1,char* s2); //最长公
共字串
**********************************************
********************/
struct SuffixArray {
    int r[maxn];
    int sa[maxn], rank[maxn], height[maxn];
    int t[maxn], t2[maxn], c[maxn], n;
    int m;//模板长度
    void init(int s[], int n) {
        this->n = n;
        for (int i = 0; i < n; i++) r[i] = int(s[i]);
        m = 1000000 + 16;
    }
    int cmp(int *r, int a, int b, int l) {
        return r[a] == r[b] && r[a + l] == r[b + l];
    }
    /**
    字符要先转化为正整数
    待排序的字符串放在 r[]数组中，从 r[0]到 r[n-1]，
    长度为 n，且最大值小于 m。
    所有的 r[i]都大于 0,r[n]无意义算法中置 0
    函数结束后，结果放在 sa[]数组中(名次从 1..n)，
    从 sa[1]到 sa[n]。s[0]无意义
    **/
    void build_sa() {
        int i, k, p, *x = t, *y = t2;
        r[n++] = 0;
        for (i = 0; i < m; i++) c[i] = 0;
        for (i = 0; i < n; i++) c[x[i] = r[i]]++;
        for (i = 1; i < m; i++) c[i] += c[i - 1];
        for (i = n - 1; i >= 0; i--) sa[--c[x[i]]] = i;
        for (k = 1, p = 1; k < n; k *= 2, m = p) {
            for (p = 0, i = n - k; i < n; i++) y[p++] = i;
            for (i = 0; i < n; i++) if (sa[i] >= k)
                    y[p++] = sa[i] - k;
            for (i = 0; i < m; i++) c[i] = 0;
            for (i = 0; i < n; i++) c[x[y[i]]]++;
            for (i = 1; i < m; i++) c[i] += c[i - 1];
            for (i = n - 1; i >= 0; i--) sa[--c[x[y[i]]]] = y[i];
            swap(x, y);
            p = 1;
            x[sa[0]] = 0;
            for (i = 1; i < n; i++)
                x[sa[i]] = cmp(y, sa[i - 1], sa[i], k) ? p - 1 : p++;
        }
        n--;
    }
    /**
    height[2..n]:height[i]保存的是 lcp(sa[i],sa[i-1])
    rank[0..n-1]:rank[i]保存的是原串中 suffix[i]的名
    次
    **/
    void getHeight() {
        int i, j, k = 0;
        for (i = 1; i <= n; i++) rank[sa[i]] = i;
        for (i = 0; i < n; i++) {
            if (k) k--;
            j = sa[rank[i] - 1];
            while (r[i + k] == r[j + k]) k++;
            height[rank[i]] = k;
        }
    }
    int d[maxn][20];
//元素从 1 编号到 n
    void RMQ_init(int A[], int n) {
        for (int i = 1; i <= n; i++) d[i][0] = A[i];
        for (int j = 1; (1 << j) <= n; j++)
            for (int i = 1; i + (1 << (j - 1)) <= n; i++)

                d[i][j] = min(d[i][j - 1], d[i + (1 << (j - 1))][j - 1]);
    }
    int RMQ(int L, int R) {
        int k = 0;
        L = rank[L];
        R = rank[R];
        if(L > R) swap(L, R);
        L++;
        while ((1 << (k + 1)) <= R - L + 1) k++;
        return min(d[L][k], d[R - (1 << k) + 1][k]);
    }
    void LCP_init() {
        RMQ_init(height, n);
    }
    int lcp(int i, int j) {
        if (rank[i] > rank[j]) swap(i, j);
        return RMQ(rank[i] + 1, rank[j]);
    }
    void call_fun(int s[], int n) {
        init(s, n);//初始化后缀数组
        build_sa();//构造后缀数组 sa
        getHeight();//计算 height 与 rank
        LCP_init();//初始化 RMQ
    }
    int so(int n) {
        int ans = 0;
        for(int i = 1; i <= n; i++) {
            ans += (n - sa[i] - height[i]);
        }
        return ans;
    }
} solver;

int a[maxn];
int n, k;

bool fuck(int x) {
    int sum = 0;
    for(int i = 2; i <= n; ++i) {
        if(solver.height[i] >= x) ++sum;
        else {
            if(sum + 1 >= k) return true;
            sum = 0;
        }
    }
    if(sum + 1 >= k) return true;
    return false;
}

int main(int argc, char **argv) {
    while(~scanf("%d%d", &n, &k)) {
        for(int i = 0; i < n; ++i) scanf("%d", &a[i]);
        solver.call_fun(a, n);
        int res = 0, L = 0, R = n;
        while(L <= R) {
            int mid = L + R >> 1;
            if(fuck(mid)) res = mid, L = mid + 1;
            else R = mid - 1;
        }
        printf("%d\n", res);
    }
    return 0;
}
```